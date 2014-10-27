---
layout: post
title: "Authenticate Web API using Access Tokens"
date: 2014-04-15 22:52:30 +0000
comments: true
categories: [WebApi, C#, AspNet]
---

In a common Web API scenario, you may want to secure your endpoints so that certain actions can only be executed by authenticated users who posses the correct permissions and are authorised to access the endpoints. For example, you would not want expose your DELETE endpoint of a resource to the general public.
<!--more-->

This problem is traditionally solved by Authentication and Authorization and your credentials are passed along with the request to the server. An alternative is to exchange the username and password for a short-lived access token and use this access token to perform the protected actions. This solution does not fit every scenario,  however, it means that if the access token gets exposed, the user credentials are not revealed. 

Authentication and Authorisation

There is a key difference between authentication and authorisation. Authentication is regarded as identifying the user – confirming that you are, who you say you are. Authorisation is a secondary step that happens once the authenticity was established. Authentication is all about “does this person have access to access this resource or perform certain action.

Creating an Access Token

The first thing the user needs to do is exchange the user credentials for an access token. This token is stored on the client side and verified every time by the service API.

In the action below, username and password is captured by the login model and passed down to the authentication service. If the auth service returns a valid user, we can create a short-lived auth token for the user to use. The token is then set in a cookie and returned as part of the response to the user.

{% highlight ruby %}
[HttpPost]
public ActionResult Index(LoginModel model)
{
    if(!ModelState.IsValid)
        return View(model);
 
    var result = _authenticationService.Authenticate(model);
    if (!result.IsAuthenticated)
        return View(model);
 
    var token = new AccessToken(result.User.Id);
    _accessTokenRepository.Save(token);
    Response.Cookies.Add(new HttpCookie("token", token.Id) { Expires = token.Expires, Path = "/" });
 
    return RedirectToAction("Index", "Security");
}
{% endhighlight %}
 

Authentication Handler to check the Access Token

Once you have created your access token, it will be sent to the server every time as part of the cookie collection. The server should then check the access token cookie on every request create an appropriate IPrincipal based on the access token.

The best way to ensure access token is processed on every request, you can create a custom handler for authentication by inheriting from DelegatingHandler class.

?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
{
    var accessToken = request.Headers.GetCookies("token");
    if (accessToken.Count == 0)
        return base.SendAsync(request, cancellationToken);
 
    var tokenValue = accessToken[0]["token"].Value;
    var token = _accessTokenRepository.FindById(tokenValue);
    if(token == null)
        return base.SendAsync(request, cancellationToken);
 
    var user = _userRepository.FindById(token.UserId);
 
    var identity = new GenericIdentity(user.Username, "Basic");
    var principal = new GenericPrincipal(identity, user.Roles.ToArray());
    Thread.CurrentPrincipal = principal;
 
    return base.SendAsync(request, cancellationToken);
}
The handler gets the access token from the cookie and attempts to find the user based on the access token. If the user is found, new GenericIdentity and GenericPrincipal are created based on the user and user’s roles. You can then assign the GenericPrincipal to the current thread.

Remember to add your AuthenticationHandler to the MessageHandlers in the Web API  GlobalConfiguration

?
1
2
3
4
5
....
GlobalConfiguration.Configuration.MessageHandlers.Add(
    new AuthenticationHandler(Container.Resolve<IAccessTokenRepository>(),
                              Container.Resolve<IUserRepository>()));
...
 

Protect API Actions with Authorize Attribute

Once the user gets authenticated and the user roles are stored on the thread’s IPrincipal, you can you use the in-built Web API Authorize attribute to check whether the user is authenticated. You can even specify roles that user needs to be in to perform a specific action.

?
1
2
3
4
5
6
7
8
9
10
11
[Authorize]
public override IEnumerable<Contact> Get()
{
    return base.Get();
}
 
[Authorize(Roles = "Administrator")]
public override System.Net.Http.HttpResponseMessage Delete(Guid id)
{
    return base.Delete(id);
}

If the user is not authenticated or does not have the correct permissions, the server will return 401 HTTP status code.

Code Sample

You can check out all the the above in the code sample on GitHub. Run the solution and navigate to /security - have a look on the SecurityController, SecureContactController and AuthenticationHandler controller. If you have any questions give me a shout @mirajavora