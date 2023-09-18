---
layout: post
title: The case of the disappearing digits
author: Lorentz Vedeler
date: 2023-09-17
tags:   
    - Misc
    - Development
---

![Black and white graphic of computers][cover_graphic ]{: height=320px width=320px}

One of the developers on my team was experiencing an issue with an authentication service. The call to the authentication service consistently failed on his machine, but it worked as expected for all the other members of the team. It also had worked for years on his previous computer. 

Upon further investigation, we discovered that when authenticating from the new computer, named 'company-firstname-win2', the digit '2' was mysteriously removed from the hostname in the request. This prompted us to wonder if there was some validation code responsible for stripping numbers from valid machine names in our codebase.

However, our search yielded no such validation code. The real culprit turned out to be the use of `Environment.MachineName` during authentication. This property returns the NetBIOS name of the computer. It just so happened that 'companyfirstname-win2' was precisely 16 characters long. While Microsoft permits longer computer names, the NetBIOS name is restricted to 15 characters, causing truncation.

To resolve this issue, we opted to use `Dns.GetHostName()` instead.


[cover_graphic]: /assets/imgs/computer_noir.png "Black and white graphic of computers"