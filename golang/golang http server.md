
First, make the `projects` directory and navigate to it:

```bash
mkdir projects
cd projects
```

Next, make the directory for your project and navigate to that directory. In this case, use the directory `httpserver`:

```bash
mkdir httpserver
cd httpserver
```

Now that you have your program’s directory created and you’re in the `httpserver` directory, you can start implementing your HTTP server.

## Listening for Requests and Serving Responses

A Go HTTP server includes two major components: the server that listens for requests coming from HTTP clients and one or more request handlers that will respond to those requests. In this section, you’ll start by using the function `http.HandleFunc` to tell the server which function to call to handle a request to the server. Then, you’ll use the `http.ListenAndServe` function to start the server and tell it to listen for new HTTP requests and then serve them using the handler functions you set up.

Now, inside the `httpserver` directory you created, use `nano`, or your favorite editor, to open the `main.go` file:

```bash
nano main.go
```

In the `main.go` file, you will create two functions, `getRoot` and `getHello`, to act as your handler functions. Then, you’ll create a `main` function and use it to set up your request handlers with the `http.HandleFunc` function by passing it the `/` path for the `getRoot` handler function and the `/hello` path for the `getHello` handler function. Once you’ve set up your handlers, call the `http.ListenAndServe` function to start the server and listen for requests.

Add the following code to the file to start your program and set up the handlers:

main.go

```go
package main

import (
    "errors"
    "fmt"
    "io"
    "net/http"
    "os"
)

func getRoot(w http.ResponseWriter, r *http.Request) {
    fmt.Printf("got / request\n")
    io.WriteString(w, "This is my website!\n")
}
func getHello(w http.ResponseWriter, r *http.Request) {
    fmt.Printf("got /hello request\n")
    io.WriteString(w, "Hello, HTTP!\n")
}
```

In this first chunk of code, you set up the `package` for your Go program, `import` the required packages for your program, and create two functions: the `getRoot` function and the `getHello` function. Both of these functions have the same function signature, where they accept the same arguments: an `http.ResponseWriter` value and an `*http.Request` value. This function signature is used for HTTP handler functions and is defined as [`http.HandlerFunc`](https://pkg.go.dev/net/http#HandlerFunc). When a request is made to the server, it sets up these two values with information about the request being made and then calls the handler function with those values.

In an `http.HandlerFunc`, the [`http.ResponseWriter`](https://pkg.go.dev/net/http#ResponseWriter) value (named `w` in your handlers) is used to control the response information being written back to the client that made the request, such as the body of the response or the status code. Then, the [`*http.Request`](https://pkg.go.dev/net/http#Request) value (named `r` in your handlers) is used to get information about the request that came into the server, such as the body being sent in the case of a `POST` request or information about the client that made the request.

For now, in both of your HTTP handlers, you use `fmt.Printf` to print when a request comes in for the handler function, then you use the `http.ResponseWriter` to send some text to the response body. The `http.ResponseWriter` is an [`io.Writer`](https://pkg.go.dev/io#Writer), which means you can use anything capable of writing to that interface to write to the response body. In this case, you’re using the [`io.WriteString`](https://pkg.go.dev/io#WriteString) function to write your response to the body.

While `http.HandlerFunc` is a convenient way to use a plain function as a request handler, it’s worth understanding the `http.Handler` interface it’s built on. The `http.Handler` interface is defined as any type that implements a single method:

```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

Any `struct` that has a `ServeHTTP` method with this signature satisfies the interface and can be used anywhere Go’s HTTP package expects a handler, including as the second argument to `http.ListenAndServe`. This becomes useful when your handler needs to carry state, such as a database connection or a configuration value, that you don’t want to pass through a global variable:

```
type helloHandler struct {
    db *sql.DB
}

func (h *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello from a handler with a DB connection!")
}
```

`http.HandlerFunc`, by contrast, is a shortcut. It is itself a type that implements `http.Handler`, but it lets you treat any function with the right signature as a handler without defining a new `struct`. Under the hood, its `ServeHTTP` method simply calls the function itself. So when you write:

```go
mux.HandleFunc("/", getRoot)
```

Go is converting your `getRoot` function into an `http.HandlerFunc` value and registering it as an `http.Handler` on the mux. The two approaches produce the same result; the distinction is one of convenience versus structure. For simple handlers, `http.HandlerFunc` keeps the code concise. For handlers that need to share state or be tested in isolation, implementing `http.Handler` directly on a struct is the cleaner choice.

Now, continue creating your program by starting your `main` function:

main.go

```go
func main() {
    http.HandleFunc("/", getRoot)
    http.HandleFunc("/hello", getHello)
    
    err := http.ListenAndServe(":3333", nil)
```

In the `main` function, you have two calls to the `http.HandleFunc` function. Each call to the function sets up a handler function for a specific request path in the default server multiplexer. The server multiplexer is an [`http.Handler`](https://pkg.go.dev/net/http#Handler) that is able to look at a request path and call a given handler function associated with that path. So, in your program, you’re telling the default server multiplexer to call the `getRoot` function when someone requests the `/` path and the `getHello` function when someone requests the `/hello` path.

Once the handlers are set up, you call the `http.ListenAndServe` function, which tells the global HTTP server to listen for incoming requests on a specific port with an optional `http.Handler`. In your program, you tell the server to listen on `":3333"`. By not specifying an IP address before the colon, the server will listen on every IP address associated with your computer, and it will listen on port `3333`. A [network port](https://en.wikipedia.org/wiki/Port_\(computer_networking\)), such as `3333` here, is a way for one computer to have many programs communicating with each other at the same time. Each program uses its own port, so when a client connects to a specific port the computer knows which program to send it to. If you wanted to only allow connections to `localhost`, the hostname for IP address `127.0.0.1`, you could instead say `127.0.0.1:3333`.

Your `http.ListenAndServe` function also passes a `nil` value for the `http.Handler` parameter. This tells the `ListenAndServe` function that you want to use the default server multiplexer and not the one you’ve set up.

The `ListenAndServe` is a blocking call, which means your program won’t continue running until after `ListenAndServe` finishes running. However, `ListenAndServe` won’t finish running until your program finishes running or the HTTP server is told to shut down. Even though `ListenAndServe` is blocking and your program doesn’t include a way to shut down the server, it’s still important to include error handling because there are a few ways calling `ListenAndServe` can fail. So, add error handling to your `ListenAndServe` in the `main` function as shown:

main.go

```go
func main() {
    ...
    err := http.ListenAndServe(":3333", nil)

    if errors.Is(err, http.ErrServerClosed) {
        fmt.Printf("server closed\n")
    } else if err != nil {
        fmt.Printf("error starting server: %s\n", err)
        os.Exit(1)
    }
}
```

The first error you’re checking for, [`http.ErrServerClosed`](https://pkg.go.dev/net/http#pkg-variables), is returned when the server is told to shut down or close. This is usually an expected error because you’ll be shutting down the server yourself, but it can also be used to show why the server stopped in the output. In the second error check, you check for any other error. If this happens, it will print the error to the screen and then exit the program with an error code of `1` using the `os.Exit` function.

One error you may see while running your program is the `address already in use` error. This error can be returned when `ListenAndServe` is unable to listen on the address or port you’ve provided because another program is already using it. Sometimes this can happen if the port is commonly used and another program on your computer is using it, but it can also happen if you run multiple copies of your own program multiple times. If you see this error as you’re working on this tutorial, make sure you’ve stopped your program from a previous step before running it again.

**Note:** If you see the `address already in use` error and you don’t have another copy of your program running, it could mean some other program is using it. If this happens, wherever you see `3333` mentioned in this tutorial, change it to another number above 1024 and below 65535, such as `3334`, and try again. If you still see the error you may need to keep trying to find a port that isn’t being used. Once you find a port that works, use that going forward for all your commands in this tutorial.

Now that your code is ready, save your `main.go` file and run your program using `go run`. Unlike other Go programs you may have written, this program won’t exit right away on its own. Once you run the program, continue to the next commands:

```bash
go run main.go
```

Since your program is still running in your terminal, you will need to open a second terminal to interact with your server. When you see commands or output with the same color as the command below, it means to run it in this second terminal.

In this second terminal, use the [curl](https://curl.se/) program to make an HTTP request to your HTTP server. `curl` is a utility commonly installed by default on many systems that can make requests to servers of various types. For this tutorial, you’ll be using it to make HTTP requests. Your server is listening for connections on your computer’s port `3333`, so you’ll want to make your request to `localhost` on that same port:

```bash
curl http://localhost:3333
```

The output will look like this:

```bash
OutputThis is my website!
```

In the output you’ll see the `This is my website!` response from the `getRoot` function, because you accessed the `/` path on your HTTP server.

Now, in that same terminal, make a request to the same host and port, but add the `/hello` path to the end of your `curl` command:

```bash
curl http://localhost:3333/hello
```

Your output will look like this:

```bash
OutputHello, HTTP!
```

This time you’ll see the `Hello, HTTP!` response from the `getHello` function.

If you refer back to the terminal you have your HTTP server function running in, you now have two lines of output from your server. One for the `/` request and another for the `/hello` request:

```bash
Outputgot / request
got /hello request
```

Since the server will continue running until the program finishes running, you’ll need to stop it yourself. To do this, press `CONTROL+C` to send your program the interrupt signal to stop it.

In this section, you created an HTTP server program, but it’s using the default server multiplexer and default HTTP server. Using default, or global, values can lead to bugs that are hard to duplicate because multiple parts of your program could be updating them at different and varying times. If this leads to an incorrect state, it can be hard to track down the bug because it might only exist if certain functions are called in a particular order. So, to avoid this problem, you’ll update your server to use a server multiplexer you create yourself in the next section.

## Multiplexing Request Handlers

When you started your HTTP server in the last section, you passed the `ListenAndServe` function a `nil` value for the `http.Handler` parameter because you were using the default server multiplexer. Because `http.Handler` is an [interface](https://www.digitalocean.com/community/tutorials/how-to-use-interfaces-in-go), it’s possible to create your own `struct` that implements the interface. But sometimes, you only need a basic `http.Handler` that calls a single function for a specific request path, like the default server multiplexer. In this section, you will update your program to use [`http.ServeMux`](https://pkg.go.dev/net/http#ServeMux), a server multiplexer and `http.Handler` implementation provided by the `net/http` package, which you can use for these cases.

The `http.ServeMux` `struct` can be configured the same as the default server multiplexer, so there aren’t many updates you need to make to your program to start using your own instead of a global default. To update your program to use an `http.ServeMux`, open your `main.go` file again and update your program to use your own `http.ServeMux`:

main.go

```go
...

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello)

    err := http.ListenAndServe(":3333", mux)

    ...
}
```

In this update, you created a new `http.ServeMux` using the `http.NewServeMux` constructor and assigned it to the `mux` variable. After that, you only needed to update the `http.HandleFunc` calls to use the `mux` variable instead of calling the `http` package. Finally, you updated the call to `http.ListenAndServe` to provide it the `http.Handler` you created (`mux`) instead of a `nil` value.

Now you can run your program again using `go run`:

```bash
go run main.go
```

Your program will continue running as it did last time, so you’ll need to run commands to interact with the server in another terminal. First, use `curl` to request the `/` path again:

```bash
curl http://localhost:3333
```

The output will look like the following:

```bash
OutputThis is my website!
```

You’ll see this output is the same as before.

Next, run the same command from before for the `/hello` path:

```bash
curl http://localhost:3333/hello
```

The output will look like this:

```bash
OutputHello, HTTP!
```

The output for this path is the same as before as well.

Finally, if you refer back to the original terminal you’ll see the output for both the `/` and `/hello` requests as before:

```bash
Outputgot / request
got /hello request
```

The update you made to the program is functionally the same, but this time you’re using your own `http.Handler` instead of the default one.

Finally, press `CONTROL+C` again to exit your server program.

### ServeMux and Third-Party Routers

In the above section, you updated your program to use an `http.ServeMux` instead of the default server multiplexer. The `ServeMux` type provided by the `net/http` package is a simple and effective way to route incoming HTTP requests to handler functions based on the request path.

The `ServeMux` works by matching the URL path of an incoming request to a registered pattern and then calling the corresponding handler. Because it is part of the Go standard library, it integrates seamlessly with the `http.Server` type and requires no additional dependencies. This makes it a strong choice for small applications and for learning how HTTP routing works in Go.

However, `ServeMux` is intentionally minimal in its design. It supports:

- exact path matching
- simple prefix-based routing

For many applications, this level of functionality is sufficient. As your application grows, though, you may encounter routing requirements that are more complex.

One common example is the need for dynamic or parameterized routes. Consider a URL such as:

```bash
/users/42
```

In this case, the value `42` represents a user ID. With `ServeMux`, handling this type of route typically requires manually parsing the URL path inside the handler. While this approach works, it can become difficult to manage as the number of routes increases.

To address these limitations, many developers turn to third-party routers that provide more advanced routing capabilities. Popular options in the Go ecosystem include:

- `chi`
- `gorilla/mux`
- `echo`
- `gin`

These routers extend the functionality of `ServeMux` and commonly support features such as:

- parameterized routes (for example, `/users/{id}`)
- route grouping and nesting
- more flexible request matching rules
- built-in helpers for applying middleware

For instance, a router may allow you to define a route like `/users/{id}` and automatically extract the value of `id` from the request URL. This reduces the need for manual parsing and keeps handler logic focused on processing the request.

Another advantage of many third-party routers is their support for middleware. While middleware can be implemented using the `http.Handler` interface, as shown earlier in this tutorial, routers often provide convenient ways to apply middleware across groups of routes or the entire application.

Despite these additional features, most third-party routers are designed to integrate with the standard `net/http` package rather than replace it. They typically implement the `http.Handler` interface, which means they can be used with `http.ListenAndServe` or an `http.Server` in the same way as a `ServeMux`.

This compatibility allows you to adopt a router incrementally. In many cases, switching from `ServeMux` to a third-party router only requires replacing the multiplexer while leaving the rest of your server configuration unchanged.

For learning purposes and simple applications, `http.ServeMux` remains an excellent starting point because it introduces routing concepts without additional abstraction. As your application grows and your routing needs become more complex, you can evaluate whether a third-party router would simplify your implementation and improve maintainability.

By understanding both approaches, you can choose the routing solution that best fits your application while continuing to build on the same core concepts provided by Go’s HTTP server.

## Running Multiple Servers at One Time

In addition to using your own `http.Handler`, the Go `net/http` package also allows you to use an HTTP server other than the default one. Sometimes you may want to customize how the server runs, or you may want to run multiple HTTP servers in the same program at once. For example, you may have a public website and a private admin website you want to run from the same program. Since you can only have one default HTTP server, you wouldn’t be able to do this with the default one. In this section, you will update your program to use two [`http.Server`](https://pkg.go.dev/net/http#Server) values provided by the `net/http` package for cases like these — when you want more control over the server or need multiple servers at the same time.

In your `main.go` file, you will set up multiple HTTP servers using `http.Server`. You’ll also update your handler functions to access the [`context.Context`](https://pkg.go.dev/context#Context) for the incoming `*http.Request`. This will allow you to set which server the request is coming from in a `context.Context` variable, so you can print the server in the handler function’s output.

Open your `main.go` file again and update it as shown:

main.go

```go
package main

import (
    // Note: Also remove the 'os' import.
    "context"
    "errors"
    "fmt"
    "io"
    "net"
    "net/http"
)

const keyServerAddr = "serverAddr"

func getRoot(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    fmt.Printf("%s: got / request\n", ctx.Value(keyServerAddr))
    io.WriteString(w, "This is my website!\n")
}
func getHello(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    fmt.Printf("%s: got /hello request\n", ctx.Value(keyServerAddr))
    io.WriteString(w, "Hello, HTTP!\n")
}
```

In the code update above, you updated the `import` statement to include the packages needed for the update. Then, you created a `const` `string` value called `keyServerAddr` to act as the key for the HTTP server’s address value in the `http.Request` context. Lastly, you updated both your `getRoot` and `getHello` functions to access the `http.Request` ’s `context.Context` value. Once you have the value, you include the HTTP server’s address in the `fmt.Printf` output so you can see which of the two servers handled the HTTP request.

Now, start updating your `main` function by adding the first of your two `http.Server` values:

main.go

```go
...
func main() {
    ...
    mux.HandleFunc("/hello", getHello)

    ctx, cancelCtx := context.WithCancel(context.Background())
    serverOne := &http.Server{
        Addr:    ":3333",
        Handler: mux,
        BaseContext: func(l net.Listener) context.Context {
            ctx = context.WithValue(ctx, keyServerAddr, l.Addr().String())
            return ctx
        },
    }
```

In the updated code, the first thing you did is create a new `context.Context` value with an available function, `cancelCtx`, to cancel the context. Then, you define your `serverOne` `http.Server` value. This value is very similar to the HTTP server you’ve already been using, but instead of passing the address and the handler to the `http.ListenAndServe` function, you set them as the `Addr` and `Handler` values of the `http.Server`.

One other change is adding a `BaseContext` function. `BaseContext` is a way to change parts of the `context.Context` that handler functions receive when they call the `Context` method of `*http.Request`. In the case of your program, you’re adding the address the server is listening on (`l.Addr().String()`) to the context with the key `serverAddr`, which will then be printed to the handler function’s output.

Next, define your second server, `serverTwo`:

main.go

```go
...

func main() {
    ...
    serverOne := &http.Server {
        ...
    }

    serverTwo := &http.Server{
        Addr:    ":4444",
        Handler: mux,
        BaseContext: func(l net.Listener) context.Context {
            ctx = context.WithValue(ctx, keyServerAddr, l.Addr().String())
            return ctx
        },
    }
```

This server is defined the same way as the first server, except instead of `:3333` for the `Addr` field, you set it to `:4444`. This way, one server will be listening for connections on port `3333` and the second server will listen on port `4444`.

Now, update your program to start the first server, `serverOne`, in a goroutine:

main.go

```go
...

func main() {
    ...
    serverTwo := &http.Server {
        ...
    }

    go func() {
        err := serverOne.ListenAndServe()
        if errors.Is(err, http.ErrServerClosed) {
            fmt.Printf("server one closed\n")
        } else if err != nil {
            fmt.Printf("error listening for server one: %s\n", err)
        }
        cancelCtx()
    }()
```

Inside the goroutine, you start the server with `ListenAndServe`, the same as you have before, but this time you don’t need to provide parameters to the function like you did with `http.ListenAndServe` because the `http.Server` values have already been configured. Then, you do the same error handling as before. At the end of the function, you call `cancelCtx` to cancel the context being provided to the HTTP handlers and both server `BaseContext` functions. This way, if the server ends for some reason, the context will end as well.

Finally, update your program to start the second server in a goroutine as well:

main.go

```go
...

func main() {
    ...
    go func() {
        ...
    }()
    go func() {
        err := serverTwo.ListenAndServe()
        if errors.Is(err, http.ErrServerClosed) {
            fmt.Printf("server two closed\n")
        } else if err != nil {
            fmt.Printf("error listening for server two: %s\n", err)
        }
        cancelCtx()
    }()

    <-ctx.Done()
}
```

This goroutine is functionally the same as the first one, it just starts `serverTwo` instead of `serverOne`. This update also includes the end of the `main` function where you read from the `ctx.Done` channel before returning from the `main` function. This ensures that your program will stay running until either of the server goroutines ends and `cancelCtx` is called. Once the context is over, your program will exit.

Save and close the file when you’re done.

Run your server using the `go run` command:

```bash
go run main.go
```

Your program will continue running again, so in your second terminal run the `curl` commands to request the `/` path and the `/hello` path from the server listening on `3333`, the same as previous requests:

```bash
curl http://localhost:3333
curl http://localhost:3333/hello
```

The output will look like this:

```bash
OutputThis is my website!
Hello, HTTP!
```

In the output you’ll see the same responses you saw before.

Now, run those same commands again, but this time use port `4444`, the one that corresponds to `serverTwo` in your program:

```bash
curl http://localhost:4444
curl http://localhost:4444/hello
```

The output will look like the following:

```bash
OutputThis is my website!
Hello, HTTP!
```

You’ll see the same output for these requests as you did for the requests on port `3333` being served by `serverOne`.

Finally, look back at the original terminal where your server is running:

```bash
Output[::]:3333: got / request
[::]:3333: got /hello request
[::]:4444: got / request
[::]:4444: got /hello request
```

The output looks similar to what you saw before, but this time it shows the server that responded to the request. The first two requests show they came from the server listening on port `3333` (`serverOne`), and the second two requests came from the server listening on port `4444` (`serverTwo`). These are the values retrieved from the `BaseContext` ’s `serverAddr` value.

Your output may also be slightly different than the output above depending on whether your computer is set up to use [IPv6](https://en.wikipedia.org/wiki/IPv6) or not. If it is, you’ll see the same output as above. If not, you’ll see `0.0.0.0` instead of `[::]`. The reason for this is that your computer will communicate with itself over IPv6 if configured, and `[::]` is the IPv6 notation for `0.0.0.0`.

Once you’re done, use `CONTROL+C` again to stop the server.

In this section, you created a new HTTP server program using `http.HandleFunc` and `http.ListenAndServe` to run and configure the default server. Then, you updated it to use an `http.ServeMux` for the `http.Handler` instead of the default server multiplexer. Finally, you updated your program to use `http.Server` to run multiple HTTP servers in the same program.

While you have an HTTP server running now, it’s not very interactive. You can add new paths that it responds to, but there’s not really a way for users to interact with it past that. The HTTP protocol includes a number of ways users can interact with an HTTP server beyond paths. In the next section, you’ll update your program to support the first of them: query string values.

## Using Middleware to Process Requests

As your HTTP server grows, you may notice that several handler functions begin to share common behavior. For instance, you might want to log details about every request, add headers to each response, or perform validation before a handler runs. If you add this logic directly into each handler function, your code can quickly become repetitive and harder to maintain.

To address this, Go applications commonly use a pattern called [**middleware**](https://drstearns.github.io/tutorials/gomiddleware/). Middleware allows you to wrap an existing handler with additional functionality so that the shared logic is executed automatically for every request. This pattern works well in Go because all HTTP handlers follow the same `http.Handler` interface introduced earlier in this tutorial.

### Creating a Middleware Function

In Go, middleware is typically written as a function that accepts an `http.Handler` and returns another `http.Handler`. This returned handler can run code before and after calling the original handler, allowing you to insert additional behavior into the request lifecycle.

To see how this works, open your `main.go` file and add the following function:

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Printf("started %s %s\n", r.Method, r.URL.Path)

        next.ServeHTTP(w, r)

        fmt.Printf("completed %s %s\n", r.Method, r.URL.Path)
    })
}
```

This function wraps another handler, referred to as `next`, and returns a new handler. When a request is received, the middleware first prints a message indicating that the request has started. It then calls `next.ServeHTTP(w, r)` to pass control to the original handler. After the handler finishes processing the request, the middleware prints another message indicating that the request has completed.

The call to `next.ServeHTTP` is what allows the request to continue through the handler chain. If this call were omitted, the request would never reach your handler function, and no response would be generated.

### Applying Middleware to Your Server

Once the middleware function has been defined, you can apply it to your server by wrapping your existing handler or multiplexer. If you are using an `http.ServeMux`, update your `main` function as shown below:

```
func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello) 

    loggedMux := loggingMiddleware(mux)

    err := http.ListenAndServe(":3333", loggedMux)
    if err != nil {
        fmt.Printf("error starting server: %s\n", err)
    }
}
```

In this example, the `mux` variable still handles routing requests to the appropriate handler functions. However, instead of passing `mux` directly to `http.ListenAndServe`, it is wrapped using `loggingMiddleware`. This means every request will pass through the middleware before reaching the handler functions you defined earlier.

If you run your server again and make a request using `curl`, you will now see additional output in your terminal showing when each request begins and ends. This confirms that the middleware is being executed for every incoming request.

### Chaining Multiple Middleware Functions

Middleware becomes even more useful when you combine multiple middleware functions together. Each middleware can wrap another, forming a chain of processing steps. For example, you might want to add a custom response header in addition to logging requests. You can define another middleware function like this:

```
func headerMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("X-App-Version", "1.0")

        next.ServeHTTP(w, r)
    })
}
```

This middleware sets a response header before passing the request to the next handler. To apply both middleware functions, you can wrap them around your handler in sequence:

```
handler := loggingMiddleware(headerMiddleware(mux))
http.ListenAndServe(":3333", handler)
```

In this setup, each request first passes through the logging middleware, then through the header middleware, and finally reaches the handler associated with the requested route. Once the handler finishes, control returns back through the middleware in reverse order.

This layered approach allows you to add shared behavior to your application without modifying each handler individually. Middleware is commonly used in [Go applications](https://www.digitalocean.com/community/tutorials/building-go-applications-for-different-operating-systems-and-architectures) to handle concerns such as logging, authentication, and request validation. Because it is built on top of the `http.Handler` interface, it works seamlessly with both the standard library and third-party routers.

By structuring your application in this way, your handler functions can remain focused on generating responses, while middleware takes care of reusable logic that applies across multiple routes.

## Inspecting a Request’s Query String

One of the ways a user is able to influence the HTTP response they get back from an HTTP server is by using the [query string](https://en.wikipedia.org/wiki/Query_string). The query string is a set of values added to the end of a URL. It starts with a `?` character, with additional values added using `&` as a delimiter. Query string values are commonly used as a way to filter or customize the results an HTTP server sends as a response. For example, one server may use a `results` value to allow a user to specify something like `results=10` to say they’d like to see 10 items in their list of results.

In this section, you will update your `getRoot` handler function to use its `*http.Request` value to access query string values and print them to the output.

First, open your `main.go` file and update the `getRoot` function to access the query string with the `r.URL.Query` method. Then, update the `main` method to remove `serverTwo` and all its associated code since you’ll no longer need it:

main.go

```go

func getRoot(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    hasFirst := r.URL.Query().Has("first")
    first := r.URL.Query().Get("first")
    hasSecond := r.URL.Query().Has("second")
    second := r.URL.Query().Get("second")

    fmt.Printf("%s: got / request. first(%t)=%s, second(%t)=%s\n",
        ctx.Value(keyServerAddr),
        hasFirst, first,
        hasSecond, second)
    io.WriteString(w, "This is my website!\n")
}
```

In the `getRoot` function, you use the `r.URL` field of `getRoot` ’s `*http.Request` to access properties about the URL being requested. Then you use the `Query` method of the `r.URL` field to access the query string values of the request. Once you’re accessing the query string values, there are two methods you can use to interact with the data. The `Has` method returns a `bool` value specifying whether the query string has a value with the key provided, such as `first`. Then, the `Get` method returns a `string` with the value of the key provided.

In theory, you could always use the `Get` method to retrieve query string values because it will always return either the actual value for the given key or an empty string if the key doesn’t exist. For many uses, this is good enough — but in some cases, you may want to know the difference between a user providing an empty value or not providing a value at all. Depending on your use case, you may want to know whether a user has provided a `filter` value of nothing, or if they didn’t provide a `filter` at all. If they provided a `filter` value of nothing you may want to treat it as “don’t show me anything,” whereas not providing a `filter` value would mean “show me everything.” Using `Has` and `Get` will allow you to tell the difference between these two cases.

In your `getRoot` function, you also updated the output to show the `Has` and `Get` values for both `first` and `second` query string values.

Now, update your `main` function to go back to using one server again:

main.go

```go
...

func main() {
    ...
    mux.HandleFunc("/hello", getHello)

    ctx := context.Background()
    server := &http.Server{
        Addr:    ":3333",
        Handler: mux,
        BaseContext: func(l net.Listener) context.Context {
            ctx = context.WithValue(ctx, keyServerAddr, l.Addr().String())
            return ctx
        },
    }

    err := server.ListenAndServe()
    if errors.Is(err, http.ErrServerClosed) {
        fmt.Printf("server closed\n")
    } else if err != nil {
        fmt.Printf("error listening for server: %s\n", err)
    }
}
```

In the `main` function, you removed references to `serverTwo` and moved running the `server` (formerly `serverOne`) out of a goroutine and into the `main` function, similar to how you were running `http.ListenAndServe` earlier. You could also change it back to `http.ListenAndServe` instead of using an `http.Server` value since you only have one server running again, but by using `http.Server`, you would have less to update if you wanted to make any additional customizations to the server in the future.

Now, once you’ve saved your changes, use `go run` to run your program again:

```bash
go run main.go
```

Your server will start running again, so swap back to your second terminal to run a `curl` command with a query string. In this command you’ll need to surround your URL with single quotes (`'`), otherwise your terminal’s shell may interpret the `&` symbol in the query string as the “run this command in the background” feature that many shells include. In the URL, include a value of `first=1` for `first`, and `second=` for `second`:

```bash
curl 'http://localhost:3333?first=1&second='
```

The output will look like this:

```bash
OutputThis is my website!
```

You’ll see the output from the `curl` command hasn’t changed from previous requests.

If you switch back to your server program’s output, however, you’ll see the new output includes the query string values:

```bash
Output[::]:3333: got / request. first(true)=1, second(true)=
```

The output for the `first` query string value shows the `Has` method returned `true` because `first` has a value, and also that `Get` returned the value of `1`. The output for `second` shows that `Has` returned `true` because `second` was included, but the `Get` method didn’t return anything other than an empty string. You can also try making different requests by adding and removing `first` and `second` or setting different values to see how it changes the output from those functions.

Once you’re finished, press `CONTROL+C` to stop your server.

In this section you updated your program to use only one `http.Server` again, but you also added support for reading `first` and `second` values from the query string for the `getRoot` handler function.

Using the query string isn’t the only way for users to provide input to an HTTP server, though. Another common way to send data to a server is by including data in the request’s body. In the next section, you’ll update your program to read a request’s body from the `*http.Request` data.

## Reading a Request Body

When creating an HTTP-based API, such as a [REST API](https://en.wikipedia.org/wiki/Representational_state_transfer), a user may need to send more data than can be included in URL length limits, or your page may need to receive data that isn’t about how the data should be interpreted, such as filters or result limits. In these cases, it’s common to include data in the request’s body and to send it with either a `POST` or a `PUT` HTTP request.

In a Go `http.HandlerFunc`, the `*http.Request` value is used to access information about the incoming request, and it also includes a way to access the request’s body with the `Body` field. In this section, you will update your `getRoot` handler function to read the request’s body.

To update your `getRoot` method, open your `main.go` file and update it to use `io.ReadAll` to read the `r.Body` request field:

main.go

```go
package main

import (
    ...
    "io"
    ...
)

...

func getRoot(w http.ResponseWriter, r *http.Request) {
    ...
    second := r.URL.Query().Get("second")

    body, err := io.ReadAll(r.Body)
    if err != nil {
        fmt.Printf("could not read body: %s\n", err)
    }

    fmt.Printf("%s: got / request. first(%t)=%s, second(%t)=%s, body:\n%s\n",
        ctx.Value(keyServerAddr),
        hasFirst, first,
        hasSecond, second,
        body)
    io.WriteString(w, "This is my website!\n")
}

...
```

In this update, you use the `io.ReadAll` function to read the `r.Body` property of the `*http.Request` to access the request’s body. The `io.ReadAll` function is a utility function that will read data from an [`io.Reader`](https://pkg.go.dev/io#Reader) until it encounters an error or the end of the data. Since `r.Body` is an `io.Reader`, you’re able to use it to read the body. Once you’ve read the body, you also updated the `fmt.Printf` to print it to the output.

After you’ve saved your updates, run your server using the `go run` command:

```bash
go run main.go
```

Since the server will continue running until you stop it, go to your other terminal to make a `POST` request using `curl` with the `-X POST` option and a body using the `-d` option. You can also use the `first` and `second` query string values from before as well:

```bash
curl -X POST -d 'This is the body' 'http://localhost:3333?first=1&second='
```

Your output will look like the following:

```bash
OutputThis is my website!
```

The output from your handler function is the same, but you’ll see your server logs have been updated again:

```bash
Output[::]:3333: got / request. first(true)=1, second(true)=, body:
This is the body
```

In the server logs, you’ll see the query string values from before, but now you’ll also see the `This is the body` data the `curl` command sent.

Now, stop the server by pressing `CONTROL+C`.

In this section, you updated your program to read a request’s body into a variable you printed to the output. By combining reading the body this way with other functionality, such as [`encoding/json`](https://pkg.go.dev/encoding/json) to unmarshal a JSON body into Go data, you’ll be able to create APIs your users can interact with in a way they’re familiar with from other APIs.

Not all data sent from a user is in the form of an API, though. Many websites have forms they ask their users to fill out, so in the next section, you’ll update your program to read form data in addition to the request body and query string you already have.

## Retrieving Form Data

For a long time, sending data using forms was the standard way for users to send data to an HTTP server and interact with a website. Forms aren’t as popular now as they were in the past, but they still have many uses as a way for users to submit data to a website. The `*http.Request` value in `http.HandlerFunc` also provides a way to access this data, similar to how it provides access to the query string and request body. In this section, you’ll update your `getHello` program to receive a user’s name from a form and respond back to them with their name.

Open your `main.go` and update the `getHello` function to use the `PostFormValue` method of `*http.Request`:

main.go

```go
...

func getHello(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    fmt.Printf("%s: got /hello request\n", ctx.Value(keyServerAddr))

    myName := r.PostFormValue("myName")
    if myName == "" {
        myName = "HTTP"
    }
    io.WriteString(w, fmt.Sprintf("Hello, %s!\n", myName))
}

...
```

Now in your `getHello` function, you’re reading the form values posted to your handler function and looking for a value named `myName`. If the value isn’t found or the value found is an empty string, you set the `myName` variable to a default value of `HTTP` so the page doesn’t display an empty name. Then, you updated the output to the user to display the name they sent, or `HTTP` if they didn’t send a name.

To run your server with these updates, save your changes and run it using `go run`:

```bash
go run main.go
```

Now, in your second terminal, use `curl` with the `-X POST` option to the `/hello` URL, but this time instead of using `-d` to provide a data body, use the `-F 'myName=Sammy'` option to provide form data with a `myName` field with the value `Sammy`:

```bash
curl -X POST -F 'myName=Sammy' 'http://localhost:3333/hello'
```

The output will look like this:

```bash
OutputHello, Sammy!
```

In the output above, you’ll see the expected `Hello, Sammy!` greeting because the form you sent with `curl` said `myName` is `Sammy`.

The `r.PostFormValue` method you’re using in the `getHello` function to retrieve the `myName` form value is a special method that only includes values posted from a form in the body of a request. However, an `r.FormValue` method is also available that includes both the form body and any values in the query string. So, if you used `r.FormValue("myName")` you could also remove the `-F` option and include `myName=Sammy` in the query string to see `Sammy` returned as well. If you did that without the change to `r.FormValue`, though, you’d see the default `HTTP` response for the name. Being careful about where you’re retrieving these values from can avoid potential conflicts in names or bugs that are hard to track down. It’s useful to be more strict and use the `r.PostFormValue` unless you want the flexibility to put it in the query string as well.

If you look back at your server logs you’ll see the `/hello` request was logged similar to previous requests:

```bash
Output[::]:3333: got /hello request
```

To stop the server, press `CONTROL+C`.

In this section, you updated your `getHello` handler function to read a name from form data posted to the page and then return that name to the user.

At this point in your program, a few things can go wrong when handling a request, and your users wouldn’t be notified. In the next section, you’ll update your handler functions to return HTTP status codes and headers.

## Responding with Headers and a Status Code

The HTTP protocol uses a few features that users don’t normally see to send data to help browsers or servers communicate. One of these features is called the [status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes), and is used by the server to give an HTTP client a better idea of whether the server considers the request successful, or whether something went wrong on either the server side, or with the request the client sent.

Another way HTTP servers and clients communicate is by using [header fields](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields). A header field is a key and value that either a client or server will send to the other to let them know about themselves. There are many headers that are pre-defined by the HTTP protocol, such as `Accept`, which a client uses to tell the server the type of data it can accept and understand. It’s also possible to define your own by prefixing them with `x-` and then the rest of a name.

In this section, you’ll update your program to make the `myName` form field of `getHello`, a required field. If a value isn’t sent for the `myName` field, your server will send back a “Bad Request” status code to the client and add an `x-missing-field` header to let the client know which field was missing.

To add this feature to your program, open your `main.go` file one last time and add the validation check to the `getHello` handler function:

main.go

```go
...

func getHello(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    fmt.Printf("%s: got /hello request\n", ctx.Value(keyServerAddr))

    myName := r.PostFormValue("myName")
    if myName == "" {
        w.Header().Set("x-missing-field", "myName")
        w.WriteHeader(http.StatusBadRequest)
        return
    }
    io.WriteString(w, fmt.Sprintf("Hello, %s!\n", myName))
}

...
```

In this update, when `myName` is an empty string, instead of setting a default name of `HTTP`, you send the client an error message instead. First, you use the `w.Header().Set` method to set an `x-missing-field` header with a value of `myName` in the response HTTP headers. Then, you use the `w.WriteHeader` method to write any response headers, as well as a “Bad Request” status code, to the client. Finally, it will return out of the handler function. You want to make sure you do this so you don’t accidentally write a `Hello, !` response to the client in addition to the error information.

It’s also important to be sure you’re setting headers and sending the status code in the right order. In an HTTP request or response, all the headers must be sent before the body is sent to the client, so any requests to update `w.Header()` must be done before `w.WriteHeader` is called. Once `w.WriteHeader` is called, the status of the page is sent with all the headers and only the body can be written to after.

Once you’ve saved your updates, you can run your program again with the `go run` command:

```bash
go run main.go
```

Now, use your second terminal to make another `curl -X POST` request to the `/hello` URL, but don’t include the `-F` to send form data. You’ll also want to include the `-v` option to tell `curl` to show verbose output so you can see all the headers and output for the request:

```bash
curl -v -X POST 'http://localhost:3333/hello'
```

This time in the output, you’ll see a lot more information as the request is processed because of the verbose output:

```bash
Output*   Trying ::1:3333...
* Connected to localhost (::1) port 3333 (#0)
> POST /hello HTTP/1.1
> Host: localhost:3333
> User-Agent: curl/7.77.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 400 Bad Request
< X-Missing-Field: myName
< Date: Wed, 02 Mar 2022 03:51:54 GMT
< Content-Length: 0
< 
* Connection #0 to host localhost left intact
```

The first couple of lines in the output show that `curl` is trying to connect to `localhost` port `3333`.

Then, the lines that start with `>` show the request `curl` is making to the server. It says `curl` is making a `POST` request to the `/hello` URL using the HTTP 1.1 protocol, as well as a few other headers. Then, it sends an empty body as shown by the empty `>` line.

Once `curl` sends the request, you can see the response it receives from the server with the `<` prefix. The first line says that the server responded with a `Bad Request`, which is also known as a 400 status code. Then, you can see the `X-Missing-Field` header you set is included with a value of `myName`. After sending a few additional headers, the request finishes without sending any of the body, which can be seen by the `Content-Length` (or body) being length `0`.

If you look back at your server output once more, you’ll see the `/hello` request the server handled in the output:

```bash
Output[::]:3333: got /hello request
```

One last time, press `CONTROL+C` to stop your server.

In this section, you updated your HTTP server to add validation to the `/hello` form input. If a name isn’t sent as part of the form, you used `w.Header().Set` to set a header to send back to the client. Once the header is set, you used `w.WriteHeader` to write the headers to the client, as well as a status code indicating to the client it was a bad request.

## Building a Simple JSON API Endpoint

In many real-world applications, HTTP servers are used to expose APIs that allow clients to send and receive structured data. These APIs are commonly referred to as JSON APIs because they use JSON as the format for both requests and responses.

Unlike earlier examples in this tutorial, where the server returned plain text, a JSON API is designed to communicate data in a structured way. This allows different systems such as web applications, mobile apps, or other services, to interact with your server programmatically.

In this section, you will create a simple JSON API endpoint that accepts data from a client, processes it, and returns a structured JSON response.

### Defining the API Request Structure

To begin, you need to define the structure of the data your API will accept. In Go, this is typically done using a `struct`.

Open your `main.go` file and add the following definition:

```go
type Message struct {
    Name string \`json:"name"\`
    Body string \`json:"body"\`
}
```

This struct represents the expected format of the incoming JSON request. For example, a client might send data like this:

```json
{
  "name": "Sammy",
  "body": "Hello from JSON"
}
```

The struct tags (such as `json:"name"`) ensure that the JSON keys are correctly mapped to the corresponding struct fields.

### Creating the JSON API Handler

Next, create a handler function that will act as your API endpoint:

```go
func handleJSON(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
        return
    }

    var msg Message

    err := json.NewDecoder(r.Body).Decode(&msg)
    if err != nil {
        http.Error(w, "invalid JSON payload", http.StatusBadRequest)
        return
    }

    response := map[string]string{
        "status":  "success",
        "message": "Received message from " + msg.Name,
        "body":    msg.Body,
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)

    json.NewEncoder(w).Encode(response)
}
```

This handler demonstrates a common pattern used in JSON APIs.

It begins by checking the HTTP method of the request. Since this endpoint is designed to accept data, it only allows `POST` requests. If a client sends a request using a different method, the server responds with a `405 Method Not Allowed` status.

Next, the handler reads and decodes the JSON payload from the request body into the `Message` struct. If the JSON is malformed or does not match the expected structure, the server returns a `400 Bad Request` response.

After successfully decoding the request, the handler constructs a JSON response. In this example, the response includes a status field and echoes part of the input data. In a real API, this is where you would typically perform application logic such as storing data or querying a database.

Before sending the response, the handler sets the `Content-Type` header to `application/json` and explicitly writes a `200 OK` status. The response is then encoded as JSON and written to the client.

### Registering the API Route

Once the handler is defined, register it in your `main` function:

```
mux.HandleFunc("/api/message", handleJSON)
```

This creates a dedicated API endpoint at `/api/message`. Using a path prefix such as `/api` is a common convention that helps distinguish API routes from other types of endpoints.

### Testing the JSON API

With your server running, you can test the API using `curl`:

```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"name":"Sammy","body":"Hello from JSON"}' \
http://localhost:3333/api/message
```

If the request is successful, the server will return a JSON response similar to the following:

```json
{
  "status": "success",
  "message": "Received message from Sammy",
  "body": "Hello from JSON"
}
```

If you send an invalid request—for example, by using the wrong HTTP method or malformed JSON—the server will respond with an appropriate HTTP status code and error message.

### Understanding the API Flow

When a client interacts with this endpoint, the communication follows a structured request-response cycle.

The client sends a POST request containing JSON data. The server reads and validates the request, processes the input, and then returns a structured JSON response along with an appropriate HTTP status code.

This pattern forms the foundation of most REST-style APIs. By following it, your server can communicate reliably with a wide range of clients while maintaining a clear and predictable interface.

By extending your HTTP server to support JSON APIs, you move beyond serving simple responses and begin building services that can integrate with other systems. This is a fundamental step in developing modern backend applications with Go.

## Production Considerations

Throughout this tutorial, you have been building HTTP servers suitable for development and learning purposes. However, when deploying an HTTP server to a production environment, there are several important considerations that can improve the reliability, security, and performance of your application.

In this section, you will learn about configuring timeouts, implementing graceful shutdown, enabling HTTPS, and applying other production-ready practices to your Go HTTP server.

### Configuring Server Timeouts

By default, Go’s HTTP server does not enforce timeouts on connections. This means a slow or malicious client could potentially hold a connection open indefinitely, consuming server resources and potentially leading to resource exhaustion.

To prevent this, you should configure explicit timeouts on your `http.Server`. Go provides several timeout settings that control different phases of the request lifecycle:

- **ReadTimeout** — The maximum duration for reading the entire request, including the body
- **WriteTimeout** — The maximum duration before timing out writes of the response
- **IdleTimeout** — The maximum amount of time to wait for the next request when keep-alives are enabled

Open your `main.go` file and update your server configuration to include these timeouts:

```go
func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello)

    server := &http.Server{
        Addr:         ":3333",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    }

    fmt.Println("Server starting on :3333")
    err := server.ListenAndServe()
    if err != nil {
        fmt.Printf("error starting server: %s\n", err)
    }
}
```

In this configuration, the server will wait a maximum of 10 seconds to read a request and 10 seconds to write a response. If a connection is idle (no active request), it will be closed after 120 seconds. These values should be adjusted based on your application’s specific needs. For example, if your server handles large file uploads, you may need to increase `ReadTimeout`.

It’s important to understand the difference between these timeout values. `ReadTimeout` covers the entire duration from when the connection is accepted to when the request body has been fully read. This means it includes the time taken to read request headers as well. `WriteTimeout` normally covers the time from the end of the request header read to the end of the response write, but this can vary based on whether the handler reads the request body.

For applications that handle long-running requests, such as streaming responses or webhooks, you may need to adjust these timeouts or implement more sophisticated timeout handling within your handler functions using the request’s context.

### Implementing Graceful Shutdown

When you need to stop or restart your server, abruptly terminating active connections can result in failed requests and a poor user experience. A graceful shutdown allows the server to stop accepting new connections while completing any in-flight requests before fully shutting down.

Go’s `http.Server` includes a `Shutdown` method that enables graceful shutdown. This method blocks until all active connections have been closed or until a provided context is canceled.

Update your `main.go` file as follows to implement graceful shutdown:

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello)

    server := &http.Server{
        Addr:         ":3333",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    }

    // Start server in a goroutine
    go func() {
        fmt.Println("Server starting on :3333")
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            fmt.Printf("error starting server: %s\n", err)
            os.Exit(1)
        }
    }()

    // Set up channel to listen for interrupt signals
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    
    // Block until signal is received
    <-quit
    fmt.Println("Server is shutting down...")

    // Create a context with timeout for shutdown
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Attempt graceful shutdown
    if err := server.Shutdown(ctx); err != nil {
        fmt.Printf("Server forced to shutdown: %s\n", err)
    }

    fmt.Println("Server stopped")
}
```

In this implementation, the server starts in a goroutine so that the `main` function can continue executing. You then set up a channel to listen for interrupt signals such as `SIGINT` (sent when you press `CONTROL+C`) or `SIGTERM` (commonly sent by process managers or orchestration systems).

When a shutdown signal is received, the program creates a context with a 30-second timeout and calls `server.Shutdown`. This tells the server to stop accepting new connections and to wait for existing requests to complete. If all requests finish within the 30-second window, the server shuts down cleanly. If the timeout is reached, the server is forced to close any remaining connections.

This pattern is particularly important in production environments where your server may be managed by a process supervisor, container orchestrator like Kubernetes, or a service manager like systemd. These systems expect applications to handle shutdown signals properly.

### Enabling HTTPS and TLS

In production environments, you should always serve traffic over HTTPS to encrypt data in transit and protect user privacy. Go’s HTTP server includes built-in support for TLS (Transport Layer Security) through the `ListenAndServeTLS` method.

To enable HTTPS, you need a TLS certificate and a private key. For testing purposes, you can create a self-signed certificate, but for production use you should obtain a certificate from a trusted Certificate Authority (CA) such as [Let’s Encrypt](https://letsencrypt.org/).

Here’s how to configure your server to use HTTPS:

```go
func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello)

    server := &http.Server{
        Addr:         ":8443",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    }

    fmt.Println("Server starting on https://localhost:8443")
    err := server.ListenAndServeTLS("cert.pem", "key.pem")
    if err != nil {
        fmt.Printf("error starting server: %s\n", err)
    }
}
```

In this example, the server listens on port `8443` (a common port for HTTPS) and uses the certificate file `cert.pem` and private key file `key.pem`. Note that the default HTTPS port is `443`, but using ports below `1024` typically requires administrator or root privileges.

When using HTTPS, you should also configure your server to use secure TLS settings. Go’s default TLS configuration is generally secure, but you can customize it for your specific requirements:

```
import (
    "crypto/tls"
    "net/http"
    "time"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello)

    tlsConfig := &tls.Config{
        MinVersion:               tls.VersionTLS12,
        PreferServerCipherSuites: true,
    }

    server := &http.Server{
        Addr:         ":8443",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
        TLSConfig:    tlsConfig,
    }

    fmt.Println("Server starting on https://localhost:8443")
    err := server.ListenAndServeTLS("cert.pem", "key.pem")
    if err != nil {
        fmt.Printf("error starting server: %s\n", err)
    }
}
```

This configuration sets the minimum TLS version to 1.2, which disables older, less secure versions of the protocol. The `PreferServerCipherSuites` setting tells the server to prefer its own cipher suite preferences over the client’s, allowing you to enforce the use of strong cipher suites.

### Adding Security Headers

Security headers are HTTP response headers that instruct browsers to enable various security protections. While these headers don’t prevent attacks on your server directly, they help protect your users by telling their browsers how to handle your content safely.

You can add security headers to your responses using middleware:

```go
func securityHeadersMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Prevent clickjacking attacks
        w.Header().Set("X-Frame-Options", "DENY")
        
        // Prevent MIME type sniffing
        w.Header().Set("X-Content-Type-Options", "nosniff")
        
        // Enable XSS protection
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        
        // Enforce HTTPS
        w.Header().Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
        
        // Control what resources can be loaded
        w.Header().Set("Content-Security-Policy", "default-src 'self'")
        
        next.ServeHTTP(w, r)
    })
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello)

    // Wrap your mux with security headers
    securedMux := securityHeadersMiddleware(mux)

    server := &http.Server{
        Addr:    ":3333",
        Handler: securedMux,
    }

    server.ListenAndServe()
}
```

Each of these headers serves a specific security purpose:

- `X-Frame-Options` prevents your site from being embedded in an iframe, protecting against clickjacking attacks
- `X-Content-Type-Options` prevents browsers from MIME-sniffing a response away from the declared content type
- `X-XSS-Protection` enables the browser’s built-in cross-site scripting filter
- `Strict-Transport-Security` tells browsers to only access your site over HTTPS
- `Content-Security-Policy` restricts what resources (scripts, styles, images) can be loaded

You should customize these headers based on your application’s specific requirements. For example, if your application needs to be embedded in iframes, you would adjust or remove the `X-Frame-Options` header.

### Implementing Health Checks

Health check endpoints allow monitoring systems, load balancers, and orchestration platforms to verify that your server is running and ready to handle requests. A basic health check is a simple endpoint that returns a successful status when the server is operational.

Add a health check endpoint to your server:

```go
func healthCheck(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    w.Write([]byte(\`{"status":"healthy"}\`))
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", getRoot)
    mux.HandleFunc("/hello", getHello)
    mux.HandleFunc("/health", healthCheck)

    server := &http.Server{
        Addr:    ":3333",
        Handler: mux,
    }

    server.ListenAndServe()
}
```

This basic health check always returns a successful response. In a more sophisticated implementation, you might check the status of dependencies such as database connections or external services before responding:

```go
func healthCheck(w http.ResponseWriter, r *http.Request) {
    // Check database connection
    if err := db.Ping(); err != nil {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusServiceUnavailable)
        w.Write([]byte(\`{"status":"unhealthy","reason":"database connection failed"}\`))
        return
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    w.Write([]byte(\`{"status":"healthy"}\`))
}
```

This enhanced version checks whether the database connection is working before declaring the service healthy. Many production systems distinguish between liveness checks (is the application running?) and readiness checks (is the application ready to serve traffic?). You can implement separate endpoints for each concern.

### Logging and Error Handling

Proper logging is essential for debugging issues in production environments. While the examples in this tutorial have used `fmt.Printf` for simplicity, production applications should use a structured logging library that supports log levels, timestamps, and structured data.

Here’s an example of a logging middleware that records information about each request:

```go
import (
    "log"
    "time"
)

func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Call the next handler
        next.ServeHTTP(w, r)
        
        // Log request details
        log.Printf(
            "%s %s %s %s",
            r.Method,
            r.RequestURI,
            r.RemoteAddr,
            time.Since(start),
        )
    })
}
```

This middleware logs the HTTP method, request URI, client address, and request duration for each request. For production use, consider using a structured logging library like `zap` or `logrus` that can output logs in JSON format for easier parsing and analysis by log aggregation systems.

You should also implement proper error handling throughout your application. Instead of printing errors to the console, log them with appropriate context, and return meaningful error messages to clients without exposing sensitive internal details.

### Rate Limiting

Rate limiting protects your server from abuse by restricting the number of requests a client can make within a given time window. While implementing a full rate limiting solution is beyond the scope of this tutorial, you can use third-party packages like `golang.org/x/time/rate` or integrate with external rate limiting services.

A basic rate limiting middleware might look like this:

```go
import (
    "golang.org/x/time/rate"
    "net/http"
)

func rateLimitMiddleware(next http.Handler) http.Handler {
    limiter := rate.NewLimiter(10, 20) // 10 requests per second, burst of 20

    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            http.Error(w, "rate limit exceeded", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

This example creates a global rate limiter that allows 10 requests per second with a burst capacity of 20 requests. In a production system, you would typically want to implement per-client rate limiting based on IP addresses or authentication tokens rather than a single global limit.

### Performance Optimization

For high-traffic production environments, you may need to optimize your server’s performance. Go’s HTTP server is already quite performant by default, but there are several techniques you can use to improve performance further:

First, ensure your handler functions are efficient and avoid unnecessary work. Use connection pooling for database connections, cache frequently accessed data, and minimize memory allocations in hot code paths.

Second, consider enabling HTTP/2, which Go’s server supports automatically when using TLS. HTTP/2 can improve performance through features like request multiplexing and header compression.

Third, if your application serves static files, consider using a Content Delivery Network (CDN) or a reverse proxy like nginx to handle static content, allowing your Go server to focus on dynamic requests.

Finally, use profiling tools like `pprof` to identify performance bottlenecks in your application. Go includes excellent built-in profiling support that can help you understand where your application is spending time and using memory.

### Deployment Considerations

When deploying your Go HTTP server to production, consider using a process manager or container orchestration system to ensure your application stays running and can be easily updated. Options include:

- [**systemd**](https://www.digitalocean.com/community/tutorials/systemd-essentials-working-with-services-units-and-the-journal) — A system and service manager for Linux that can start your server automatically, restart it if it crashes, and handle logging
- [**Docker**](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04) — Containerize your application for consistent deployments across different environments
- [**Kubernetes**](https://www.digitalocean.com/community/tutorials/how-to-setup-k3s-kubernetes-cluster-on-ubuntu) — For more complex deployments requiring scaling, load balancing, and high availability

You should also implement monitoring and alerting to track your server’s health and performance over time. Services like [Prometheus](https://prometheus.io/), [Grafana](https://grafana.com/), or cloud-based monitoring solutions can help you identify issues before they impact users.

Finally, always test your production configuration thoroughly in a staging environment that mirrors your production setup as closely as possible. This helps catch configuration issues, performance problems, and potential security vulnerabilities before they affect real users.

By implementing these production considerations, you can build Go HTTP servers that are secure, reliable, and performant in real-world environments.

## FAQs

### 1\. How do I create a simple HTTP server in Go?

To create a simple HTTP server in Go, you need to use the `net/http` package from the standard library. First, define a handler function that accepts `http.ResponseWriter` and `*http.Request` parameters. Then, register this handler using `http.HandleFunc()` and start the server with `http.ListenAndServe()`:

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

This creates a server that listens on port 8080 and responds with “Hello, World!” to all requests. For production use, you should create your own `http.ServeMux` instead of using the default one and configure an `http.Server` with appropriate timeouts.

### 2\. What is the net/http package in Go?

The `net/http` package is part of Go’s standard library and provides all the functionality needed to build HTTP clients and servers. It includes types and functions for handling HTTP requests and responses, routing, cookies, headers, and more. The package implements the HTTP/1.1 and HTTP/2 protocols and integrates seamlessly with Go’s concurrency features.

Key components include `http.Server` for creating servers, `http.ServeMux` for routing requests, `http.Handler` and `http.HandlerFunc` for defining request handlers, and `http.Client` for making HTTP requests. Because it’s part of the standard library, you don’t need to install any external dependencies to build fully functional web servers in Go.

### 3\. Do I need a framework to build a web server in Go?

No, you don’t need a framework to build a web server in Go. The `net/http` package in the standard library provides everything necessary to create production-ready HTTP servers. This includes routing, middleware support through the `http.Handler` interface, request parsing, response writing, and TLS support.

However, as your application grows more complex, you may benefit from third-party routers or frameworks that provide additional features such as parameterized routes (like `/users/{id}`), easier middleware chaining, and built-in helpers for common tasks. Popular options include `chi`, `gorilla/mux`, `echo`, and `gin`. These tools are designed to work with the standard library rather than replace it, so you can adopt them incrementally as your needs evolve.

For learning and small to medium applications, the standard library is often sufficient and has the advantage of having no external dependencies.

### 4\. How do I return JSON from a Go HTTP server?

To return JSON from a Go HTTP server, use the `encoding/json` package to encode your data and set the appropriate `Content-Type` header. First, define a struct to represent your data, then use `json.NewEncoder()` to write the JSON response:

```go
import (
    "encoding/json"
    "net/http"
)

type Response struct {
    Message string \`json:"message"\`
    Status  string \`json:"status"\`
}

func handler(w http.ResponseWriter, r *http.Request) {
    response := Response{
        Message: "Success",
        Status:  "ok",
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(response)
}
```

Always set the `Content-Type` header to `application/json` before writing the response. You can also use `json.Marshal()` if you need the JSON as a byte slice before writing it. Remember to handle encoding errors appropriately in production code.

### 5\. How do I add routes in a Go web server?

In Go, you add routes by registering handler functions with a server multiplexer. The simplest approach is to use `http.HandleFunc()` to associate a URL path with a handler function:

```go
mux := http.NewServeMux()
mux.HandleFunc("/", homeHandler)
mux.HandleFunc("/about", aboutHandler)
mux.HandleFunc("/api/users", usersHandler)
```

Each call to `HandleFunc()` creates a new route. The `http.ServeMux` supports exact path matching and prefix-based routing. Paths ending with a slash match all URLs with that prefix, while paths without a trailing slash match only that exact path.

For more complex routing needs, such as URL parameters (like `/users/123`), HTTP method-based routing, or route grouping, consider using a third-party router like `chi` or `gorilla/mux`. These routers implement the same `http.Handler` interface, so they integrate seamlessly with the standard library.

### 6\. What port does a Go HTTP server run on by default?

Go HTTP servers don’t have a default port. You must explicitly specify the port when calling `http.ListenAndServe()`. The port is provided as part of the address string, such as `":8080"` or `"localhost:3000"`.

Common conventions include:

- Port 80 for HTTP (requires root/admin privileges)
- Port 443 for HTTPS (requires root/admin privileges)
- Ports 8080, 8000, or 3000 for development
- Any port above 1024 for unprivileged applications

If you specify just `":8080"`, the server listens on all available network interfaces. To restrict it to localhost, use `"127.0.0.1:8080"` or `"localhost:8080"`.

### 7\. Is Go good for building web servers?

Yes, Go is excellent for building web servers. The language was designed with network services in mind and offers several advantages:

- **Performance**: Go compiles to native machine code, resulting in fast execution and low memory usage. Go’s HTTP server can handle thousands of concurrent connections efficiently.
- **Built-in concurrency**: Go’s goroutines make it easy to handle multiple requests concurrently without the complexity of traditional threading models. Each HTTP request is automatically handled in its own goroutine.
- **Standard library**: The `net/http` package provides production-ready HTTP server functionality out of the box, including support for HTTP/2, TLS, and WebSockets.
- **Simple deployment**: Go programs compile to a single binary with no runtime dependencies, making deployment straightforward.
- **Strong typing**: Go’s static typing helps catch errors at compile time, reducing bugs in production.

Many companies use Go for high-performance web services, including Google, Uber, Dropbox, and Docker. It’s particularly well-suited for building APIs, microservices, and backend systems that need to handle significant traffic with low latency.

