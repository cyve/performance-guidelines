# HTTP security guidelines

### X-Frame-Options
Add `X-Frame-Options` header to your HTTP responses to indicate if the page can be rendered in an `<iframe>`.  
See https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options

### Content-Security-Policy
Add `Content-Security-Policy` header to your HTTP responses to indicate which assets the page is allowed to load.  
See https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP

### Same origin policy (CORS)
Restrict the origin policy to the minimum. Use header `Access-Control-Allow-*` carefully.  
See https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy

### Cookies
Restrict access to your cookies by adding `Secure`, `HttpOnly` and `SameSite` attributes.  
See https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies

### CSRF protection
Protect unsafe HTTP methods (POST, PUT, DELETE) with a CSRF token. Never use safe methods to persist data.   
See https://symfony.com/doc/current/security/csrf.html

### X-Forwarded-*
Never trust `Forwarded`, `X-Forwarded-For` or `X-Forwarded-Host` headers.  
See https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded

### HTTP Strict Transport Security
Add `Strict-Transport-Security` header to your HTTP responses to prevent the browser to load the page in HTTPS only.
See https://developer.mozilla.org/en-US/docs/Glossary/HS

### Other links
- https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks

### php.ini
```ini
; Leave function arguments out of stack traces, they can hold sensitive information
; Default: Off
zend.exception_ignore_args=On

; Don't copy process ENVVARS into $_ENV on worker start (you should use getenv() instead).
; Default: "EGPCS" (Env, Get, Post, Cookie, Server)
variables_order="GPCS"

; Add into $_REQUEST GET, then POST parameters. Exclude COOKIES, they can be set by other means.
; Default: same as variables_order
request_order="GP"

; Disable loading extensions at runtime with dl()
; Default: On
enable_dl=Off
```
