---
layout: post-with-comments
title: Your Chromium browser needs approval for Google Password Manager
category:
- til
tags:
- chromium
date: 2026-01-30 20:57 +0900
---
A few weeks ago, I worked on a Chromium-based browser on Android.

I tried signing in a website using crendential managers such as Google Password
Mangager, MS authenticator, and 1Password. However it didn't work, and I spent
quite a bit time debugging until I found the following line in logs:
```
01-21 11:58:19.051 27926 27926 E cr_ChromiumWebauthn: com.google.android.gms.common.api.ApiException: 17: API: Fido.FIDO2_PRIVILEGED_API is not available on this device. Connection failed with: ConnectionResult{statusCode=RESTRICTED_PROFILE, resolution=null, message=null, clientMethodKey=null}
```
Yes, that was it! As explained in [the official documentation][1], web browsers
are considered priviliged apps that need to handle credentials stored in
third-party applications. The browser must be a trusted app by Google Password
Manger to manage user credentials for external services.

So, how do we get this approval? No worries -- You can find the request form in
the [the official documentation][1].

[1]: https://developer.android.com/identity/sign-in/privileged-apps
