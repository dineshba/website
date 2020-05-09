---
title: "CSRF and its protection"
date: 2020-05-08T19:27:40+05:30
---

#### What is Cookie ?

An HTTP cookie (web cookie, browser cookie) is a small piece of data that a **server sends** to the user's web browser. The browser may store it and **send it back** with the next request to the same server. [source](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

> Note: It is sent to the server back by the browser **by default**

#### Cross-Site Request Forgery (CSRF)

Lets say you logged into the bank website and the bank website uses cookies to store the user authentication details (say sessionId or oauth tokens)

- Lets say the attacker sends a mail which contains hidden javascript to send ajax requests to the bank website and the victim opens it
- Lets say the attacker creates a website which will to send ajax requests to the bank website and the victim opens that website

In both these cases, browser will send the valid cookies of the bank website to it and the bank site cannot differentiate between the proper and malicious request.

#### Can this attack be applied to all kind of requests ?

Only to the **state chaging** requests which means generally POST/PUT/PATCH

##### Why it is not applicable to read-only requests ?

For read only requests, the malicious website will receive the data from the bank site. It cannot do anything with that <have some doubts here>

#### One of the easy way to protect:

##### Can javascript read cookies ?

- For `httpOnly=true` cookie (httpOnly is one of the property of the cookie), javascript can **never** read the cookie. But the browser can send it to the same server which provided it
- For `httpOnly=false` cookie, **only javascript running in the same domain can read the cookie**. Eg: cookie of `www.myback.com` can be read by the javascript running in `www.myback.com` page.

So we can create a `random string` from the backend and send to the browser as the `httpOnly=false` cookie (say, name of the cookie `CSRF-TOKEN`). We can write javascript to read the cookie and add as **header** to all the `state changing` requests (say, the header name is `X-CSRF-TOKEN`).

So our backend will receive two things in the request header.
    - `CSRF-TOKEN` sent by browser by default
    - `X-CSRF-TOKEN` sent by the javascript running in the bank website pages.

**If both are same**, then request is originated from the proper website. Simple 🥳🥳🥳