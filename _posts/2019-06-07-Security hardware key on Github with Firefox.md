---
layout: post
title: Security hardware key on Github with Firefox
subtitle: "Spoof User Agent in Firefox"
show-img: true
tags: [howto, firefox, U2F, SoloKeys, security, FIDO U2F, 2FA]
thumbnail-img: /img/posts/2019-06-07/solokey.jpg
cover-img: "/assets/img/head/security.jpg"
---
Github currently supports hardware security keys (SoloKeys, Yubico, ...) as second factor only with Google Chrome but not with Firefox,
although Firefox supports U2F and on other sites this also works flawlessly.   
> This browser doesn't support the FIDO U2F standard yet.
<img src="/img/posts/2019-06-07/github_error.jpg">  

This can be bypassed by pressenting Github a user agent from Chrome.


* call configuration page with `about:config`
* Add `general.useragent.override.github.com` as new setting
* Use `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36` as value
<img src="/img/posts/2019-06-07/github_agent.jpg">


After a reload of the site you can register your hardware security key on Github.
<img src="/img/posts/2019-06-07/github_addkey.jpg">

The entry `general.useragent.override.github.com` can be reverted to default after successful registration of the key.
<img src="/img/posts/2019-06-07/github_agent_def.jpg">

After entering the username and password on the login page you can youse your security key. 
<img src="/img/posts/2019-06-07/github_auth.jpg">
