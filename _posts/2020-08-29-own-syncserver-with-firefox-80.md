---
layout: post
title: Own syncserver with Firefox 80 (79 on Android)
show-img: true
tags: [firefox, howto, cloud]
thumbnail-img: /assets/img/cloud.png
cover-img: "/assets/img/head/cloud.jpg"
---

Since the Firefox update to version 80 and 79 on android a few things have changed to be able to use your own syncserver as described in the blog post [Setup your own Firefox Sync Server with Docker](/2018-11-03-firefox-syncserver-with-docker/).

## Desktop

In addition to the `identity.sync.tokenserver.uri` settings for the syncserver URL, the following settings must be set to `false`.

```txt
identity.fxaccounts.useSessionTokensForOAuth
identity.sync.useOAuthForSyncToken
```

<img src="../img/posts/2020-08-29/about_config.jpg">

After that you can synchronize again with your own syncserver.

## Android

The `about:config` page no longer exists in this version for Android.
So the procedure to enter the syncserver URL is now as follows:

1. Go to Settings
2. About Firefox
3. Click 5 times on to the Firefox logo until `Debug menue is enabled` appears
4. Go back to settings
5. Now you see under the Account options the option `Userdefined syncserver`, Enter your syncserver URL here.
