---
layout: post
title: Unit testing Legacy code
---

Let's get to the point. I have this nice little piece of legacy code:

{% highlight cs %}
public static void Save(object key, object value, System.DateTime? expiry, string cookieValue, bool useRequestCookieIfNew, ICookieDomainResolver domainResolver = null)
{
    if (!string.IsNullOrEmpty( key.ToString()))
    {

        var cookie = HttpContext.Current.Response.CookiesSafe(cookieValue);
        if (cookie == null || string.IsNullOrEmpty(cookie.Value))
        {
            if (useRequestCookieIfNew)
            {
                cookie = HttpContext.Current.Request.CookiesSafe(cookieValue);
            }
        }
        if (cookie == null)
        {
            cookie = new HttpCookie(cookieValue);
        }
        cookie.Values.Set(key.ToString(), value.ToString());

        cookie.Domain = domainResolver == null ? HttpContext.Current.Request.ServerVariables["HTTP_HOST"].StripSubDomains() : domainResolver.GetDomain();

        if (expiry.HasValue)
        {
            cookie.Expires = expiry.Value;
        }

        HttpContext.Current.Response.Cookies.Set(cookie);
    }
}
{% endhighlight %}

Here we are setting an Cookie, possibly considering a bunch of parameters, including setting an expiry date if provided. 

We have also this function:
{% highlight cs %}
public static HttpCookie Load(string cookieName)
{
    var cookie = HttpContext.Current.Response.CookiesSafe(cookieName);
    if (cookie == null || string.IsNullOrEmpty(cookie.Value))
    {
        cookie = HttpContext.Current.Request.CookiesSafe(cookieName);
        if (cookie == null || string.IsNullOrEmpty(cookie.Value))
        {
            return new HttpCookie(cookieName)
            {
                Domain = (HttpContext.Current.Request.ServerVariables["HTTP_HOST"] ?? "").StripSubDomains()
            };
        }
    }
    return cookie;
}
{% endhighlight %}

In this case we get the cookie given the name of it.

Not the worst code, yet not beautiful code, , easy to follow and not many responsibilities going on. There is a problem though: there aren't any unit tests covering it.
Some people may argue that it isn't a problem, for me, it is. I don't feel like I want to touch that code. It is in a shared library, if I fuck it up, I will probably make a mess, breaking some other stuff.
TDD is about workflow, Unit testing is about confidence. If I don't have any unit tests around it, my confidence is, zero?

We all have had to change some code like this. In my case I have to provide a parameter so that when creating the cookie, it can be created as an [httponly or not](https://www.owasp.org/index.php/HttpOnly).
I'm planning to pass an optional parameter, **isHttpOnly**, and then doing something like: **cookie.HttpOnly =  isHttpOnly**.
Simple code, only an assigment. I'd really like to unit test my change, but this code is difficult to unit test.
The big deal is this line:

{% highlight cs %}
var cookie = HttpContext.Current.Response.CookiesSafe(cookieValue);
{% endhighlight %}

And this:
{% highlight cs %}
HttpContext.Current.Response.Cookies.Set(cookie);
{% endhighlight %}

Probably I would have been a lazy guy and chunk my code in it. This time around it is different, I am doing Pair programming with [Mahmut](https://twitter.com/LordSuperAstro).
He acts as an angel on my shoulder, as a good friend telling me the things I should do, even if those things are hard to hear: even if my code if simple, I want to have confidence in my change.
Most importantly I want to apply the [Boy Scout rule](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule). Next time someone else lands in this code they will find some unit tests so that the campground will get cleaner and cleaner.

Ok, let's try to unit test this code. How can I solve the problem of having the static HttpContext? How can I unit test it? 
We could create [an addapter for the HttpContext](http://tech.findmypast.com/dont-mock-what-you-dont-own/) and mock it in our unit tests.
It isn't easy in this case since the whole class is static:

{% highlight cs %}
public static class SimpleCookieHandler
{}
{% endhighlight %}

We can't inject an addapter through the constructor. We shouldn't either do method injection, that would mean introducing some breaking changes, this is a nuget package used in god knows how many places.

Ok, I'll figure this out. Let's write a unit test:

{% highlight cs %}
public class When_saving_cookie_with_httpOnly_true
{
    Because of = () =>
    {
        SimpleCookieHandler.Save("test", "", _cookieName, true, _httpContextProvider);
    };

    It should_be_httpOnly = () =>
    {
        HttpCookie cookie = SimpleCookieHandler.Load(_cookieName, _httpContextProvider);
        cookie.HttpOnly.ShouldBeTrue();
    };

    Establish context = () =>
    {
        _cookieName = "cookie-cookie";
        var response = MockRepository.GenerateMock<HttpResponseBase>();
        response.Stub(r => r.Cookies).Return(new HttpCookieCollection()
        {
            new HttpCookie(_cookieName)
            {
                HttpOnly = true
            }
        }); 

        var request = MockRepository.GenerateMock<HttpRequestBase>();
        request.Stub(r => r.Cookies).Return(new HttpCookieCollection());
        request.Stub(r => r.ServerVariables).Return(new NameValueCollection());
        var fakeHttpContext = MockRepository.GenerateMock<HttpContextBase>();
        fakeHttpContext.Stub(h => h.Response).Return(response);
        fakeHttpContext.Stub(h => h.Request).Return(request);

        _httpContextProvider = () => fakeHttpContext;
    };

    static string _cookieName;
    private static Func<HttpContextBase> _httpContextProvider;
}
{% endhighlight %}

This is the important bit:

{% highlight cs %}

SimpleCookieHandler.Save("test", "", _cookieName, true, _httpContextProvider);

/.../

HttpCookie cookie = SimpleCookieHandler.Load(_cookieName, _httpContextProvider);

/.../

private static Func<HttpContextBase> _httpContextProvider = () => fakeHttpContext;
{% endhighlight %}

Wait a second, you told me we couldn't inject anything to avoid breaking changes.
That's right, but let's go to the method:

{% highlight cs %}
internal static void Save(object key, object value, string cookieName, bool httpOnly, Func<HttpContextBase> httpContextProvider, ICookieDomainResolver domainResolver = null)
{}
{% endhighlight %}

I see... you have changed the existing method from public to internal, what about the public method?

Here you have it, in the same class:
{% highlight cs %}
public static void Save(object key, object value, string cookieName, bool httpOnly, ICookieDomainResolver domainResolver = null)
{
    Save(key, value, cookieName, httpOnly, () => new HttpContextWrapper(HttpContext.Current), domainResolver);
}
{% endhighlight %}

So if you see the public method calls the new internal method that holds all the legacy code. The public method sends an anonymous function providing the actual HttpContext.
In our test we pass an anonymous function providing the fake http context:

{% highlight cs %}
  SimpleCookieHandler.Save("test", "", _cookieName, true, () => MockRepository.GenerateMock<HttpContextBase>());
{% endhighlight %}

Now we see some tests around the legacy code. I have some confidence in my tests and the next person touching this code will find some ground where she could write her tests.

Thanks Mahmut for pushing the quality of our code and pushing me toward becoming a better developer.
Now I can send this to code review without the shame of having to explain why I didn't write any unit tests.
