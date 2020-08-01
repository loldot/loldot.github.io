---
layout: post
title: Hacktivitycon writeup - Template Shack
author: Lorentz Vedeler
date: 2020-07-31
tags:   
    - Hacktivitycon
category: writeup
---

Template Shack was a challenge worth 150 points in the 2020 Hacktivitycon CTF. The template shack appears to be some sort of online shop for bootsrap themes. The site seems to be mostly static and placeholder code. The only thing of note in the source code for the page is a comment that hints at some administration panel that is under construction.

``` html
<!-- TODO: Finish developing the admin section and add functionality to manage the templates --\>
```

![The template shack front page][template-shack]

But we can find that the site sets a cookie with the name 'token' and a value that looks like a JWT token with the typical structure of three base64 encoded parts separated by a period. I went directly to jwt.io to decode the token and found the following information in the token:

**header:**
``` json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

**body:**
``` json
{
  "username": "guest"
}
```


If you have not seen a JWT token before, it works almost like stamped ID-papers. A user is issued a token from an authorative source (usually the server or some third party login provider) who "stamps" the token with a cryptographic signature. The user can now bring his signed token and show it as proof (s)he is who (s)he claims to be. Whenever the server requires that the user authenticates, the user will pass the jwt token in the Authorize header. If the signature is valid the server will accept the users token and allow the user to proceed with the actions (s)he is authorized for. This is very useful for scaling web applicaitons as the server does not have to maintain sessions for each user, however giving the user control of the token can be a security risk if it is not implemented correctly.

Whenever I see a JWT token in a challenge I will always try to use an old trick where I create my own token with the algorithm set to none and removing the signature. In the past many libraries would accept an unsigned token in this way. That didn't work this time though.

Since the algorithm is HS256 it may be using a weak secret. I ran a script to format the token and signature in a way John the ripper can understand. Then I used john the ripper with the rockyou.txt wordlist to try and find the secret. It returns almost immediatly with the value 'supersecret'. Now we can actually just set the secret in jwt.io and craft any token we want. I chose to try setting the username to 'admin' first. Then I updated the 'token' cookie with the forged token and refreshed the page. 

![Forging the admin token in jwt.io][forged-token]

Now a link to an admin page appears. Clicking the link reveals a dashboard, where not much seems to be interactive yet either. 

![The admin panel][admin-page]

Looking a bit closer we can see that some of the pages give a rather peculiar 404 message that differs from the 404 page outside the admin panel; the url we tried to reach is printed back to us. The title of the challenge seems to hint that the page could be vulnerable to a template injection attack. When we render dynamic pages on the server it is often helpful to use a templating language that will interpolate our variables from our server code into the the html code that is sent to the client. For instance we might have a simple template that looks like this: {% raw %} `<p> {{ user.name }} </p>` {% endraw %} to print the name of the user inside a paragraph tag. Templating engines are usually very complex, so if the user is able to control the input for the templating engine that would in most cases lead to remote code execution.

To check for injection I try to insert {% raw %} `{{ 1 + 1 }}` {% endraw %} in the url and check what is written back to us. Turns out the application is in fact vulnerable to an SSTI attack. I already knew it was some python app from the http header `'Server: Werkzeug/1.0.1 Python/3.6.9'` we received from the server with every request. So enumerating to find the template engine used was just a matter of trying and failing with a few payloads. Turns out it is using the jinja templating engine, so we can find an appropriate payload to something appropriate for remote code execution in jinja.

![A successful SSTI injection][SSTI]

Looking for the flag, we first check if there is any thing of interest in the web directory:

`http://jh2i.com:50023/admin/%7B%7Brequest.application.__globals__.__builtins__.__import__('os').popen('ls -lah').read()%7D%7D`

    total 32K
    drwxr-xr-x 1 root root 4.0K Jul 25 02:12 .
    drwxr-xr-x 1 root root 4.0K Jul 25 02:06 ..
    -rw-r--r-- 1 root root   26 Jul 25 01:57 flag.txt
    -rwxr-xr-x 1 root root 4.7K Jul 25 01:57 main.py
    -rwxr-xr-x 1 root root 1.3K Jul 25 01:57 posts.py
    -rwxr-xr-x 1 root root  108 Jul 25 01:57 requirements.txt
    drwxr-xr-x 1 root root 4.0K Jul 25 01:57 templates

Found it ;) Now we just have to change our payload to read the flag.txt file:

`http://jh2i.com:50023/admin/%7B%7Brequest.application.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read()%7D%7D`


[template-shack]: /assets/imgs/template-shack.png "The template shack front page"
[forged-token]: /assets/imgs/forged-token.png "Forging a token in jwt.io"
[admin-page]: /assets/imgs/admin-page.png "The admin panel"
[SSTI]: /assets/imgs/SSTI.png "Successful SSTI"