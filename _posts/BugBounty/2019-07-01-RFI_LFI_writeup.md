---
title: "How I escalated to RFI into LFI"
date: 2019-07-01
tags: [BugBounty, BugBountyTips, Server Side Request Forgery, Local File Disclosure,]
excerpt: "How I escalated to RFI into LFI"
header:
  image: "assets/images/localfileread_banner.png"

---

Hello World, today I am going to share one of my recent interesting finding that is RFI to LFI vulnerability. It was a private program, So let's call it private.com

During my research, I came across an interesting endpoint which was taking the CSV file URL from the bucket and was including the CSV data into the site. Let's look at the endpoint first.

```console
private.com/redacted/redacted/redacted?file=https://redacted-dev.s3.amazonaws.com/file.csv
```

## Summary:

The file parameter is used to fetch the records from the CSV file. After reading data from the CSV file the data is being put to the database and data is being printed/added to the products. So basically, It just fetches the data using jquery fetch() network request and then inserts the data into the database and add the products to the private.com. So, Let's exploit this endpoint.

So, If I change the URL to my server, the application will read data from my server's CSV file and will include it into the server. Now let's jump into the ways I exploited that issue.

## EXPLOIT CASE /1 - Limited RFI

As there is no restriction in the file parameter, an attacker can use any external site to the file endpoint. The file parameter will send a request to the attacker's server and products will be added to the account without any verification here.

```console
private.com/redacted/redacted/redacted?file=https://a48e7786.ngrok.io/file.csv
```

this is called remote file inclusion since we are using external files that are hosted on external domains or any domain/site in the world.

## EXPLOIT CASE /2 - CSRF to Add products to Panel

As such there was no CSRF token in the request, So we can add CSV data into the private.com using CSRF.

```html
<html>
  <body>
    <form action="private.com/redacted/redacted/redacted?file=https://redacted.ngrok.io/file.csv">
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>

```

## EXPLOIT CASE /3 - DOS vulnerability

Thanks to @CreedHackers, for giving me the idea to change this bug into DOS vulnerability. This endpoint also leads to Application Level DOS vulnerability. Since the endpoint is used to load the files from the server-side, So We can load any file from the server. So if I open dev/random, before reading it, it will take buffer to the memory which will be huge and we will get Dos vulnerability.

```console
https://redacted.redacted.com/redacted/redacted/redacted?file=/dev/random
```

/dev/random is the Linux file that contains the environmental variables and the resources from the system. Also, that's not the regular file, the file contains many special characters while will eventually being fetched by the server. After using `?file=/dev/random`, the response of the request delayed to 67,000 millis. I did one more thing here, which I should not do, I created a wordlist of all of the Linux files and launched an intruder attack. And their server went down, I received 503 response after that xD.

<img src="https://github.com/Splint3r7/web/raw/master/assets/images/BugBountyImages/server_down.png" alt="">

## EXPLOIT CASE /4 - Blind Local file Enumeration

We can also enumerate the server for local files. If the file is available the response of the server will be:

<img src="https://raw.githubusercontent.com/Splint3r7/web/master/assets/images/BugBountyImages/local_file_enumerate_response.png" alt="">

Otherwise, the response from the server will be 404.html page. But, that's not much being used as we cannot read any file from the server we can just enumerate them blindly.

## EXPLOIT CASE /5 - SSRF

As from the exploit case (Limited RFI), we can see that we can include external urls. I tried using other schemas such as dict://, sftp://, tftp://, file://, ldap://, Gopher://, Tried few other tricks as well like gopher http, gopher http back connect but nothing seems to be working except https://. I could only do a PORT scan with it. If you want to check SSRF in details https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery is for you. Let's jump into the LFI vulnerability section since SSRF was limited and boring, wasn't able to do much with it.

<img src="https://github.com/Splint3r7/web/raw/master/assets/images/BugBountyImages/ssrf_request.png" alt="">

## EXPLOIT CASE /6 - Local File Read (Finally !!!).

After spending weeks on this program, I was still not able to escalate it to the local file. As you know from above exploit cases that the endpoint is vulnerable to RFI, SSRF and other vulnerabilities, So I was asking my self why I just cannot find the LFI straight away! Lol :).

Okay, After a week I started my research again on that program private.com. I found a strange pattern there. Okay, Let me explain that. I found that there is a pattern in every file upload functionalities. For example, if there is a profile picture upload functionality there would be an endpoint with GET request to fetch the uploaded image from the server. Similarly, there were other upload sections like pdf, CSV etc. Whenever you upload a file there's always an endpoint to fetch it with POST request. I don't know why developers coded that In this way, that seems strange. Also, I used Wayback to find that endpoint.

So, the endpoint that I have does not have upload functionality, but It made me think that if file parameter reads the CSV file and add the items into the private.com, then it should save that data somewhere somehow. I know that Looks so strange but later, I was going through the JS files of the server and I came across an endpoint that fetches the logs from the server. These logs contain the file's that are uploaded to the server and that endpoint simple fetches them.

Let's look at JS one's the endpoint:

<img src="https://github.com/Splint3r7/web/raw/master/assets/images/BugBountyImages/js_endpoint_ok.png" alt="">

Okay, it was getting bit messy, let me summarise everything that is happening there.if you look at the RFI explained above you can see that the file parameter takes CSV file and added the data from it to the site, Now, Consider, I have send file=/etc/passwd, request to the server. Now on the backend, it will create a file passwd and assign an id to it. And another endpoint that I found out from the JS file, will open up that file for me, simply :)

<img src="https://github.com/Splint3r7/web/raw/master/assets/images/BugBountyImages/LFI_csvread.png" alt="">

That's not a complete /etc/passwd file but its Enough to get P1 severity.

So, It's like developers are taking every file as CSV using csvread function, then they save that file, logs it and assigns it an ID, fetches it from another endpoint. Done! job done for me as well, thanks devs ;)

That was an interesting LFI that I wanted to share with you guys, I hope you have learned something new from it. I would be sharing more bugs, ctfs, htb_writeups in future. So, Don't forget to follow me on twitter @Splint3r7.
