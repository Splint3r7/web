---
title: "Securing Ruby on Rails Application"
date: 2021-04-02
tags: [BugBounty, BugBountyTips, Ruby on Rails, Rails]
excerpt: "Securing Ruby on Rails Application"
header:
  image: "assets/images/BugBountyImages/ror_security/ruby-on-rails-security-basics.jpg"

---

This blog post explains common vulnerabilities in ruby on rails applications with mitigations and patch recommendations.

## CSRF (Cross Site Request Forgery)

By default ruby on rails application prevents Cross-Site Request Forgery attacks using the authenticity_token in the POST request in newly created rails applications.

```
POST /posts HTTP/1.1
Host: 127.0.0.1:3000
....
....

authenticity_token=DFRt4QKVpHF1Dro2tIVsln59fAr%2BAKvVNDSHy9%2BSqc06G9BBSERVpfxT5Kh8ETFGWpjfSR2gDcmRlQzgICA%2FSg%3D%3D&post%5Btitle%5D=test&post%5Bbody%5D=test&commit=Save+Post
```

For older versions of the rails application you can manually enable the CSRF protection by introducing protect_from_forgery in your application controller.

```ruby
class PublicAppController < ActionController::Base
  protect_from_forgery
end
```

Also, this is to be noted that by default ruby on rails application does not protect an application from CSRF attacks in GET requests.

Blog Post about How I was able to invite users as admin in the organization by Exploiting [XSS and CSRF](http://hassankhanyusufzai.com/privilege-escalation-via-xss/).

## Cross Site Scripting

By default ruby on rails provides protection against cross site scripting attacks. 

default case if your print any parameter it will not be vulnerable to XSS attacks for example:

```jsx
<% if params[:name] %>
<%= params %>
```

But if you used he below-mentioned cases it, then cross-site scripting will be introduced in ruby on rails application.

XSS using content_tag:

```jsx
<% if params[:name] %>
<%= content_tag params %>
```

XSS using raw:

```jsx
<% if params[:name] %>
<%= raw params %>
```

XSS using html_safe:

```jsx
<% if params[:name] %>
<%= "<a>#{params}</a>".html_safe %>
```

### Fix Cross Site Scripting :

`sanitize` function can be used to prevent cross-site scripting attacks. But this function does not properly sanitize and has been identified as flawed numerous times. this function is also prone to HTML injection attacks.  

To fix cross-site scripting vulnerabilities use rails HTML sanitizer gem ( [https://github.com/rails/rails-html-sanitizer](https://github.com/rails/rails-html-sanitizer) ).

A sample function to sanitize tags and attributes using rails HTML sanitizer. 

```ruby
require 'rails-html-sanitizer'

Allowed_tags = %w[h1 h2 h3].freeze
Allowed_attributes = %w[style].freeze

def filter(s, tags = Allowed_tags, attributes = Allowed_attributes)

	return '' unless s

    sanitizer = Rails::Html::SafeListSanitizer.new
    string = sanitizer.sanitize(s, tags: tags, attributes: attributes)
    html_string = string.gsub(/\"/,"'")
    puts html_string
end
```

calling the filter method with user input as `filter("<h4 onclick=alert(1)>XSS</h1>")` will only return XSS string. The function will block XSS attacks but is prone to html injection.

Another gem that I recently explored is loofah. let's create a small function using loofah and blacklist HTML tags using it.

```ruby
require 'loofah'
require 'rails-html-sanitizer'

Disallowed_tags = %w[a iframe].freeze

def filter(s , disallowed_tags = Disallowed_tags)

	scrubber = Rails::Html::TargetScrubber.new
	scrubber.tags = disallowed_tags
	html = Loofah.fragment(s)
	html.scrub!(scrubber)
	f_string = html.to_s
	puts f_string

end
```

Now calling the function with user input as iframe payload:

```ruby
filter("<iframe src=x> xss testing </iframe>")
```

which will print only `xss testing`. So, this is how we can filter required tags using loofah.

Another XSS is ignored many times is found numerous times in link_to

```ruby
<%= link_to "site link", @user.site %>
```

if the @user.site contains user_input as `javascript:alert(1)` then it will trigger XSS, and link will be generated as:

```ruby
<a href="javascript:alert('1')">Site</a>
```

## SQL injections

By default, ORM ( Object Relational Mapper ) prevents the SQL injection attacks in ruby on rails application.

```ruby
User.where([“name LIKE ?”, params[:q])
```

Passing the parameter as plain SQL query will lead to SQL injection attacks.

```ruby
User.find_by_sql("name LIKE #{params[:q]}")
```

```ruby
User.where("username = #{params[:username]}")
```

SQL injection in find_by

```ruby
user = User.find_by(params[:id])
```

find_by is prone to SQL injection attacks. This can be fixed using find_by_id

```ruby
user = User.find_by(id: params[:id])
```

More examples of SQL injections in ruby on rails can be found at - [ [https://rails-sqli.org/](https://rails-sqli.org/) ] 

### Fix SQL injections:

An other solution to fix SQL injection attack in ruby on rails is to use ransack queries.

## Local File Read

Local File Read vulnerability in ruby on rails application occurs when user input is directly passed into file opening methods. for example:

```ruby
file_path = params[:user][:file_path]
file = File.open(file_path)
```

To fix local file read vulnerabilities we can create a blacklist-based method that converts the special characters into blank space.

```ruby
def s(string_)
	bad_chars = [ '/', '', ':', '"', '<', '>', '.', ' ' ]
	bad_chars.each do |x|
		string_.gsub!(x, '')
	end
	puts string_
end
```

Backlists are always prone to bypass using different encodings like URL encoding, Unicode encoding. So it's always recommended to implement a whitelist instead of a blacklist.

here's an example of a whitelist function that only allows specific pages and the rest of the user inputs are blocked. 

```ruby
def lfi_whitelist(string)
	arr = ["posts", "gallary"]
	counter = 0
	for x in arr do
		x = string
		counter += 1
		if x.match("^posts$") or x.match("^gallary$")
			puts x
		end
		break if counter==1
	end
end
```

So if you call the function function like `lfi_whitelist("../../../etc/passwd")`, the function won't return anything.

## SSRF Vulnerability in Rails Application:

SSRF vulnerability in rails applications happens if you pass user input directly in `open("user_input_url").read` function of the rails. 

```ruby
require 'sinatra'
require 'open-uri'

get '/' do
  format 'RESPONSE: %s', open(params[:url]).read
end
```

Since this will load any arbitrary URL, So we can exploit this by sending calls to AWS metadata URL and can extract identity credentials keys of the ec2 instance.

So the simple curl request to exploit this should look like:

`curl -X GET --url "localhost:4567/?url=http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]`

and you will able to extract keys:

```
RESPONSE: {
  "Code" : "Success",
  "LastUpdated" : "2021-03-22T07:54:05Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "Aredacted",
  "SecretAccessKey" : "redacted",
  "Token" : "redacted",
  "Expiration" : "2021-03-22T14:06:53Z"
}
```

### Fix SSRF vulnerabilities:

You can fix SSRF vulnerabilites using the gem [ssrf_filter](https://github.com/arkadiyt/ssrf_filter). 

```ruby
require 'ssrf_filter'

get '/' do
	response = SsrfFilter.get(params[:url])
	puts response.body
end
```

Here is the recent SSRF write-up, I have identifed where wicked pdf was being used. [SSRF to local File Read](http://hassankhanyusufzai.com/SSRF-to-LFI/)

## Command Injections

Some common command lines that are generally used in rails application are:

```ruby
eval("ruby code here")
system("os command here")
`ls -al /` # (backticks contain os command)
exec("os command here")
open("\| os command here")
```

Command injections usually occur in the rails application when user input is passed inside the above-mentioned functions. A typical example of a command injection is given below. 

```ruby

user_supplied_path = params[:cmd]
path = "#{Rails.root}/public/downloads/#{user_supplied_path}"
`ls #{path}`
```

Since this user input is directly being pass in the back-ticks, so an attacker will be able to execute just by using `test; whoami` as in user-supplied input.

### Command Injections using open3:

```html
require 'open3'
require 'sinatra'

get '/' do
	cmd = 'bash'
	user_input = params[:cmd]

	Open3.popen2e(cmd) do |stdin, stdout_stderr, wait_thread|
		Thread.new do
			stdout_stderr.each {|l| puts l }
  	end
  	stdin.puts "ls #{user_input}"
  	stdin.close

  	wait_thread.value
end
end
```

The above code snipped is vulnerable to command injection attacks. If you pass the user input `cmd=|`sleep 5`` the application response will be delayed for 5 seconds which indicates successful command injection attack.

### Fix Command Injections:

To fix command injections we need to break our command into separate strings 

```ruby
user_supplied_path = params[:cmd]
path = "#{Rails.root}/public/downloads/#{user_supplied_path}"
system("ls", path)
```

it's not recommended to fix command injection vulnerabilities using a blacklist method since they are prone to bypass using different encoding types. but if you do, then make sure you block these special characters from your user input.

```html
bad_chars = [ '/', '\\', '?', '%', '*', ':', '|', '"', '<', '>', '.', ' ', ";", "&&", "||", "`", "&", "$" ] 
```

You can tweak this array a bit based on your project requirements but make sure you blacklist `| ; & && % $ ` ||` characters from your input.

[shellescape](https://apidock.com/ruby/Shellwords/shellescape) is another method  in the rails library that can be used to fix command injection issues. That's also mentioned in [brakeman issue's](https://github.com/presidentbeef/brakeman/issues/1159) section.

## Mass Assignment:

Mass assignment is a vulnerability in rails application that allows the attacker to set model attributes by manipulating the parameter in the model attribute.

Consider a case where a user lets say manager user is able to add funds into his account.

```jsx
Class User < ActiveRecord::Base
		attr_accessible :first_name :last_name :funds
end 
```

Now consider user send's a GET request to the server to change his first name and last name from the user model, request would look like this:

```jsx
GET /users/edit?user[first_name]=hassan&user[last_name]=khan
```

Now, from the attacker's perspective, an attacker can add fund parameters and can add funds in his account without the admin's approval. 

```jsx
GET /users/edit?user[funds]=100 
```

Attacker will be able to add 100$ funds into his account.

### Fix Mass Assignment:

Use permit to permit to fix mass assignment security issues. 

```jsx
def allowed_parameters
	params.require(:user).permit(:first_name, :last_name:)
end
```

This concept is called using strong parameters in rails.

## Insecure Direct Object Reference in rails:

```ruby
@user = User.find_by(id: params[:user_id])
```

### Fix IDOR's

```ruby
@user = current_user.find_by(id: params[:user_id])
```

### Logging sensitive parameters :

By Default Ruby on Rails Application everything, including user names, passwords, credit card numbers etc. To filter those special character's make sure you initialize `Rails.application.config.filter_parameters` in your `filter_parameter_logging.rb` file.

Example:

```ruby

# Filter sensitive parameters
Rails.application.config.filter_parameters += [
  :password,
  'user.password'
]
```

## Sensitive files:

```
/config/database.yml                 -  May contain production credentials.
/config/initializers/secret_token.rb -  Contains a secret used to hash session cookie.
/db/seeds.rb                         -  May contain seed data including bootstrap admin user.
/db/development.sqlite3              -  May contain real data.
/db/seeds/users.rb                   -  May contain application default users
/config/application.yml              -  May contain env variables
/config/secrets.yml                  -  May contain application secret_key_base 
/config/yetting.yml.                 -  May contain application sensitive keys
```

If you have identified LFI or any other file read vulnerability you can extract these sensitive files to escalate your bug. Sometimes these sensitive files are not in straight /config/ folders so you have to guess the directory.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">If you are able to exploit rails CVE 5418 <br>try these things, you may end up getting the database file&#39;s credentials<br><br>config/database.yml<br>/opt/config/database.yml<br>/var/config/database.yml<br>/var/company_name/config/database.yml<br><br>The last one worked for!<a href="https://twitter.com/hashtag/BugBountyTip?src=hash&amp;ref_src=twsrc%5Etfw">#BugBountyTip</a> <a href="https://twitter.com/hashtag/BugBountyTips?src=hash&amp;ref_src=twsrc%5Etfw">#BugBountyTips</a> <a href="https://t.co/1OkYNwBJDa">pic.twitter.com/1OkYNwBJDa</a></p>&mdash; Splint3r7 (@Splint3r7_) <a href="https://twitter.com/Splint3r7_/status/1113703804552318977?ref_src=twsrc%5Etfw">April 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 


## **Black Box Enumeration of Rails Application:**

### - Wordlists

You can use multiple wordlists to enumerate an application running on ruby on rails. here's the list of wordlists:

- [Seclists - ror.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/ror.txt)

- [Rails dot files](https://secur.codes/werdlists/webapp-files/rails-dot-files.txt)

***Tip***: While running dirsearch scan on rails application always brute force with extensions .json .html .xml, sometimes the application leaks sensitive information in other file formats. 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Checking if a website is using Ruby on Rails<br><br>&gt; Search for &quot;csrf-param&quot; meta tag<br>&gt; js files with path /assets/application-*.js<br>&gt; Check their Job postings<br>&gt; _session_id in Cookie value (not a solid indicator)<br>&gt; Check for Server/X-Powered-By header</p>&mdash; Somdev Sangwan (@s0md3v) <a href="https://twitter.com/s0md3v/status/1177183347887312897?ref_src=twsrc%5Etfw">September 26, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### - Burp Suite Extension

Burp Suite Extensions that can help you in black box testing of Ruby on Rails Applications are:

- Active Scan ++
- Back Slash Power Scanner
- J2EEScan
- Attack Surface Detector

## Static Code Analysis:

### **1- Brakeman**

To identify potential security issues in your code use [brakeman](https://github.com/presidentbeef/brakeman). it will help you to identify vulnerabilities including Cross-Site Scripting, SQL injections, Command Injections, Insecure Direct Object Reference, and other OWASP security issues.

### 2- **dawnscanner**

**[dawnscanner](https://github.com/thesp0nge/dawnscanner) -** A static analysis security scanner for ruby applications.

## Vulnerabilities and Security Advisories

### 1- Bundle-audit

[Bundler-audit](https://github.com/rubysec/bundler-audit) is a gem that reads your Gemfile and based on the existing versions identifies existing flaws in gems.

### 2- ruby-advisory-db

[ruby-advisory-db](https://github.com/rubysec/ruby-advisory-db) is a database of vulnerable Ruby Gems.

### **3- GemScanner:**

GemScanner - Another tool that I have recently coded to identify older version of gems in your Gemfile.lock. The purpose of this tool is not to identify vulnerability in a gem but to find the current or latest version of gems.

[![asciicast](https://asciinema.org/a/jJYO1WP2ctszJWGVdb6MYWIJJ.svg)](https://asciinema.org/a/jJYO1WP2ctszJWGVdb6MYWIJJ)