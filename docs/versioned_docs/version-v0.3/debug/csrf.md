---
id: csrf
title: Common Cookie and CSRF Pitfalls
---

import useBaseUrl from '@docusaurus/useBaseUrl'

Because ORY Kratos is not just an API, but instead talks to your users' browsers
directly, several security measures have been implemented in ORY Kratos. One of
them is protection against CSRF:

> CSRF is an attack that tricks the victim into submitting a malicious request.
> It inherits the identity and privileges of the victim to perform an undesired
> function on the victim’s behalf. For most sites, browser requests
> automatically include any credentials associated with the site, such as the
> user’s session cookie, IP address, Windows domain credentials, and so forth.
> Therefore, if the user is currently authenticated to the site, the site will
> have no way to distinguish between the forged request sent by the victim and a
> legitimate request sent by the victim.
>
> [Source](https://owasp.org/www-community/attacks/csrf)

To protect against CSRF, several endpoints are protected by Anti-CSRF measures.
Typically, endpoints accepting `POST`, `DELTE`, `PUT` actions have Anti-CSRF
measures. When rendering a form for example, a
`<input type="hidden" name="csrf_token" value="...">` HTLM Input Element is
added. ORY Kratos compares that value to the value set in the Anti-CSRF Cookie.
If the values match, the request is allowed.

ORY Kratos uses HTTP Cookies to store login sessions when accessed via a
browser.

## Common Pitfalls

Sometimes, cookies and CSRF just wont work - all requests end up with a 401
Unauthorized or 400 Bad Request. Here are some common causes and easy fixes if
that happens to you!

Before starting to debug cookie and CSRF issues, make sure to check out the
Chrome Developer Tools (or any comparable technology) Cookies tabs in the
Application tab

<img alt="Google Chrome Developer Tools - Application Tab - Cookies"
src={useBaseUrl('img/docs/csrf_app_tab.png')} />

as well as the network tab - look for `Cookie` and `Set-Cookie` HTTP Headers:

<img alt="Google Chrome Developer Tools - Network Tab - Cookies"
src={useBaseUrl('img/docs/csrf_network_tab.png')} />

### ORY Kratos running over HTTP without dev-mode enabled

ORY Kratos' cookies have the `Secure` flag enabled by default. This means that
the browser will not send the cookie unless the URL is a HTTPS URL. If you want
ORY Kratos to work with HTTP (e.g. on localhost) you can add the `--dev` flag:
`kratos serve --dev`.

Do not do this in production!

### Running on separate (sub-)domains

Cookies work best on the same domain. While it is possible to get cookies
running on subdomains it is not possible to do that across Top Level Domains
(TLDs).

Make sure that your application (e.g. the SecureApp from the quickstart) and ORY
Krato's Public API are available on the same domain - preferably without
subdomains. Hosting both systems and routing paths with a Reverse Proxy such as
Nginx or Envoy or AWS API Gateway is the best solution. For example, routing
`https://my-website/kratos/...` to ORY Kratos and `https://my-website/dashboard`
to the SecureApp's Dashboard. Alternatively you can use piping in your app as we
do in the Quickstart guide.

We do not recommend running them on separate subdomains, e.g.
`https://kratos.my-website/` and `https://secureapp.my-website/`).

Running the apps on different TLDs will not work at all, e.g. e.g.
`https://kratos-my-website/` and `https://secureapp-my-website/`.

Running the services on different ports however is ok, if the domain stays the
same.

### Mixing up 127.0.0.1 and localhost

As already explained, make sure that the TLD stays the same. This is especially
true for `127.0.0.1` and `localhost` which are both separate TLDs. Make sure
that you use `127.0.0.1` or `localhost` consistently across your configuration!

### Trying to access ORY Kratos Cookies from client-side JavaScript

The cookies ORY Kratos sets cannot be accessed directly from Client-Side
JavaScript because the `HttpOnly` flag is set. This flag cannot be modified!

### Accessing ORY Kratos APIs from client-side JavaScript / AJAX

When implementing a Single Page App (SPA) using AngularJS or ReactJS you
probably want to access ORY Krato's Public APIs.

To prevent several attack vectors, ORY Krato's Public API is protected - even
for some GET requests - with Anti-CSRF measures.

Because AJAX does not send cookies per default, you need to configure your AJAX
request to include cookies. Using the Browser's `fetch` function, you need to
set
[`credentials: 'include'`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters)
for example.

### Accessing ORY Kratos APIs from a server-side application

When implementing a server-side application (e.g. in PHP, Java, Go, NodeJS, ...)
make sure to only call ORY Kratos' APIs through the Admin Port (default `4434`),
**not** the Public Port (default `4433`) as the Public Port requires CSRF
Cookies and the Admin Port doesn't! Since your server-side application does not
use a browser to interact with Kratos, CSRF Cookies will not be available which
causes API calls to fail.
