# Session

Before proceeding to implement the authentication function, we have to
understand a "session" first. A web application operates over HTTP
protocol which is stateless; each request and its corresponding response
is handled independently. Hence, a HTTP server cannot know whether a
series of requests is sent from the same client or from different
clients. That means the server cannot maintain client's state between
multiple requests.

Application servers maintain a *session* to keep a client's state. When
the server receives the first request from a client, the server creates
a session and give the session a unique identifier. The client should
send a request with the session identifier. The server can determine
which session the request belongs to.

In a Java EE environment, an application server creates a
`javax.servlet.http.HttpSession` object to track client's session. ZK's
`org.zkoss.zk.ui.Session` is a wrapper of
`HttpSession`, you can use it to store user's data when you handle
events. The usage:

-   Get current session: `Sessions.getCurrent()`
-   Store data into a session: `Session.setAttribute("key", data)`
-   Retrieve data from a session: `Session.getAttribute("key")`

Almost all business applications require a security mechanism and
authentication is the fundamental part of it. Some resources are only
available to those authenticated users. Authentication is a process to
verify that a user is who he claims to be. After we authenticate a user,
the application needs to remember the user and identifies his subsequent
requests so that the application doesn't need to authenticate him
repeatedly for each further request. Hence, storing user credentials in
a session is good practice and we can tell whom the request belongs to
by the request session. Additionally, we can't tell whether a request
comes from an authenticated user by checking a user's credentials in the
request's session.
