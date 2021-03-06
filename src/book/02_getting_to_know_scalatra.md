:markdown
Getting to know Scalatra
=======================

## It's Witchcraft

You saw in the introduction how to install Scalatra, its dependencies, and
write a small "hello world" application. In this chapter you will get a
whirlwind tour of the framework and familiarize yourself with its features.

## Routing

Scalatra is super flexible when it comes to routing, which is essentially an
HTTP method and a regular expression to match the requested URL. The four basic
HTTP request methods will get you a long ways:

*   GET
*   POST
*   PUT
*   DELETE

Routes are the backbone of your application, they're like a guide-map to how
users will navigate the actions you define for your application.

They also enable to you create [RESTful web services][restful-web-services], in
a very obvious manner. Here's an example of how one-such service might look:

    get("/dogs") {
      # get a listing of all the dogs
    }

    get("/dog/:id") {
      # just get one dog, you might find him like this:
      val dog = Dog.find(params("id"))
      # using the params convention, you specified in your route
    }

    post("/dog") {
      # create a new dog listing
    }

    put("/dog/:id") {
      # HTTP PUT request method to update an existing dog
    }

    delete("/dog/:id") {
      # HTTP DELETE request method to remove a dog who's been sold!
    }

As you can see from this contrived example, Scalatra's routing is very easy to get
along with. Don't be fooled, though, Scalatra can do some pretty amazing things
with Routes.

Take a more indepth look at [Scalatra's routes][routes], and see for yourself. 

[routes]: http://www.scalatra.org/book/#Routes
[restful-web-services]: http://en.wikipedia.org/wiki/Representational_State_Transfer#RESTful_web_services

## Filters

Scalatra offers a way for you too hook into the request chain of your
application via [Filters][filters].

Filters define two methods available, `before` and `after` which both accept a
block to yield corresponding the request and optionally take a URL pattern to
match to the request.

### beforeAll

The `beforeAll` method will let you pass a block to be evaluated **beforeAll** _each_
and_every_ route gets processed.

    beforeAll {
      MyDb.connect
    }

    get("/") {
      val list = MyDb.findAll()
      templateEngine.layout("index.ssp", list)
    }

In this example, we've set up a `before` filter to connect using a contrived
`MyDB` module.

### afterAll

The `afterAll` method lets you pass a block to be evaluated **afterAll** _each_ and
_every_ route gets processed.

    afterAll{
      MyDB.disconnect
    }

As you can see from this example, we're asking the `MyStore` module to
disconnect after the request has been processed.

### Pattern Matching

Filters optionally take a pattern to be matched against the requested URI
during processing. Here's a quick example you could use to run a contrived
`authenticate!` method before accessing any "admin" type requests.

    beforeSome("/admin/*") {
      basicAuth
    }
    afterSome("/admin/*") {
      user.logout
    }


[filters]: http://www.scalatra.org/book#Filters

## Handlers

Handlers are top-level methods available in Scalatra to take care of common HTTP
routines. For instance there are handlers for [halting][halting] and
[passing][passing].

There are also handlers for redirection:

    get("/"){
      redirect("/someplace/else")
    }

This will return a 302 HTTP Response to `/someplace/else`.

Scalatra has session handling built in by default. There are no modules or 
traits that you need to include.


Then you will be able to use the default cookie based session handler in your
application:

    get("/") {
      if(!session('counter')) session('counter') = 0
      session('counter') += 1
      "You've hit this page {session('counter')} times!" 
    }

Handlers can be extremely useful when used properly, probably the most common
use is the `params` convention, which gives you access to any parameters passed
in via the request object, or generated in your route pattern.

[halting]: http://www.scalatra.org/book/#Halting
[passing]: http://www.scalatra.org/book/#Passing

## Halting

To immediately stop a request within a filter or route:

    halt()

You can also specify the status:

    halt(403)

Or the status and the body:

    halt(403, <h1>Go away!</h1>)

Or even the HTTP status reason and headers.  For more complex invocations, it 
is recommended to use named arguments:

    halt(status = 403,
         reason = "Forbidden",
         headers = Map("X-Your-Mother-Was-A" -> "hamster",
                       "X-And-Your-Father-Smelt-Of" -> "Elderberries"),
         body = <h1>Go away or I shall taunt you a second time!</h1>)

The `reason` argument is ignored unless `status` is not null.  The default 
arguments leave that part of the request unchanged.

## Passing

A route can punt processing to the next matching route using pass.  Remember, unlike Sinatra, routes are matched from the bottom up.

    get("/guess/*") {
      "You missed!"
    }

    get("/guess/:who") {
      params("who") match {
        case "Frank" => pass()
        case _ => "You got me!"
      }
    }

The route block is immediately exited and control continues with the next matching route.  If no matching route is found, a 404 is returned.


## Templates

Scalatra is built upon an incredibly powerful templating engine, [Scalate][scalate].
Which, is designed to be a "thin interface" for frameworks that want to support
multiple template engines.

Some of Scalate's other all-star features include:

*   Custom template evaluation scopes / bindings
*   Ability to pass locals to template evaluation
*   Support for passing a block to template evaluation for "yield"
*   Backtraces with correct filenames and line numbers
*   Template file caching and reloading

And includes support for some of the best engines available, such as
[SSP][ssp], [SCAML][scaml], and [Jade][jade].

All you need to get started is `Scalate`, which is included in Scalatra. Views by
default look in the `views` directory in your application root.

So in your route you would have:

    get("/") {
      templateEngine.layout("index.ssp")
      # renders webapp/index.ssp
      # OR look in a sub-directory

      templateEngine.layout("/dogs/index.ssp")
      # would instead render webapp/dogs/index.ssp
    }

Another default convention of Scalatra, is the layout, which automatically looks
for a `webapp/layout` template file to render before loading any other views. In
the case of using `SSP`, your `webapp/layout.ssp` would look something like
this:

    <html>
      <head>..</head>
      <body>
        <%= yield %>
      </body>
    </html>

The possibilities are pretty much endless, here's a quick list of some of the most common use-cases covered in the README:

*   [Inline Templates][inline]
*   [Embedded Templates][embedded]
*   [Named Templates][named]

For more specific details on how Scalatra handles templates, check the [README][templates].

[inline]: http://www.scalatra.com/book/#Inline%20Templates
[embedded]: http://www.scalatra.com/book/#Embedded%20Templates
[named]: http://www.scalatra.org/book/#Named%20Templates
[templates]: http://www.scalatra.org/book/#Views%20/%20Templates

## Helpers

Helpers exist as traits in Scalatra that can applied to your base class. Please see [Helpers][helpers] for more details.

## Accessing the Servlet API

### HttpServletRequest

The request is available through the `request` variable.  The request is implicitly extended with the following methods:

1. `body`: to get the request body as a string
2. `isAjax`: to detect AJAX requests
3. `cookies` and `multiCookies`: a Map view of the request's cookies
4. Implements `scala.collection.mutable.Map` backed by request attributes

### HttpServletResponse

The response is available through the `response` variable.

### HttpSession

The session is available through the `session` variable.  The session implicitly implements `scala.collection.mutable.Map` backed by session attributes.  To avoid creating a session, it may be accessed through `sessionOption`.

### ServletContext

The servlet context is available through the `servletContext` variable.  The servlet context implicitly implements `scala.collection.mutable.Map` backed by servlet context attributes.

## Configuration

The environment is defined by:
1. The `org.scalatra.environment` system property.
2. The `org.scalatra.environment` init property.
3. A default of `development`.

If the environment starts with "dev", then `isDevelopmentMode` returns true.  This flag may be used by other modules, for example, to enable the Scalate console.

## Error handling

Error handlers run within the same context as routes and before filters.

### Not Found

Whenever no route matches, the `notFound` handler is invoked.  The default behavior is:

    notFound {
      <h1>Not found.  Bummer.</h1>
    }

* _ScalatraServlet_: send a 404 response
* _ScalatraFilter_: pass the request to the servlet filter chain