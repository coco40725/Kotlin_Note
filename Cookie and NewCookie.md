## Cookie and NewCookie

### 1. NewCookie
Used to create a new HTTP cookie, transferred in a **response**.
NewCookie, when sent in the Response, will set a ``Set-Cookie`` response header with the cookie information.
(server 端產生 Cookie 透過 response 回傳給 client)

The argument of NewCookie:
- name: cookie name
- value: cookie value
- path: the URI path for which the cookie is valid
- domain: the host domain for which the cookie is valid.
- comment: the comment.
- maxAge: the maximum age of the cookie in seconds.
- secured: specifies whether the cookie will only be sent over a secure connection. (secure connection 指的是只能透過 https 傳遞，因此 http 就不能傳遞)
- httpOnly: if true make the cookie HTTP only, i.e. only visible as part of an HTTP request.

```kotlin
@GET
@Path("/login/{name}/{email}")
@Produces(MediaType.APPLICATION_JSON)
fun login(@PathParam("name") name:String, @PathParam("email") email:String) : Response {
    val token = authService.login(email, name)
    val newCookie = NewCookie("jwt-token", token, "/", "localhost", "login token", 60*60*24, true)
    return Response.ok().cookie(newCookie).build()
}
```
Then in the Postman, you can check the **response header**, and the results would be like:
```json
{
    "Content-Type": "application/json",
    "Set-Cookie": "jwt-token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2VtYWlsIjoiYWFhQGdtYWlsLmNvbSIsInVzZXJfbmFtZSI6InRlc3QxIiwiZXhwIjoxNjgwMDU5NDU2LCJpYXQiOjE2NzkxOTU0NTZ9.YYFmlDe2BoW6JsjZC3M0f_2Z7xJu21uI9ceB2Mk8fy0;Version=1;Comment="login token";Domain=localhost;Path=/;Max-Age=86400;Secure",
    "content-length": 0
}
```
Also, in the browser, you can type the F12,  check "Application" --> "Storage" --> "Cookies", and still you would find the same results.


### 2. Cookie
Represents the value of a HTTP cookie, transferred in a request. RFC 2109 specifies the legal characters for name, value, path and domain. The default version of 1 corresponds to RFC 2109. Cookie will set the Cookie request header with the cookie information. This is per the HTTP spec.
(看起來是 client 端產生 Cookie 透過 request 傳給 server 端，但實際上不是這樣，其實是 server 端產生 cookie (NewCookie) 透過 response 傳給 client 存取，之後在特定情況 client 會再將這個 cookie 透過 request (cookie 會放在 request header) 傳給 server 端)

> This means that in the Response, you will have NewCookies and you you need to turn those into Cookies for the next request. This can easily be accomplished by calling newCookie.toCookie()

```kotlin
@GET
@Path("/logout")
fun logout(@Context headers: HttpHeaders) : Response{
    // cookie 放在 Headers，我們要拿名字是 jwt-token 的 cookie
    val cookie : Cookie? = headers.cookies?.get("jwt-token")
    return Response.ok("logout success")
        .cookie(authService.logout())
        .build()
}
```

#### Reference
https://stackoverflow.com/questions/34046292/javax-ws-rs-core-cookie-vs-javax-ws-rs-core-newcookie-what-is-the-difference

