


## 1. `http.NewServeMux()` + `http.Server`

go
```
mux := http.NewServeMux()

mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Printf("server: %s /\n", r.Method)
})

server := http.Server{
    Addr:    fmt.Sprintf(":%d", serverPort),
    Handler: mux,
}

if err := server.ListenAndServe(); err != nil {
    if !errors.Is(err, http.ErrServerClosed) {
        fmt.Printf("error running http server: %s\n", err)
    }
}
```

Here, you explicitly create a custom `ServeMux` and assign it to a server.

- `http.NewServeMux()` creates a new, independent router.
- `mux.HandleFunc()` registers routes on that router.
- `http.Server{...}` lets you configure the server.
- `server.ListenAndServe()` starts that specific server using the specified handler.

You can configure more options on `http.Server`, such as read/write timeouts, idle timeouts, TLS settings, and more.

For example:

go
```
server := &http.Server{
    Addr:         fmt.Sprintf(":%d", serverPort),
    Handler:      mux,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  60 * time.Second,
}

log.Fatal(server.ListenAndServe())
```

This is useful when you need explicit server configuration. In production, also consider graceful shutdown and whether your error handling should distinguish `http.ErrServerClosed` from other errors.

## 2. The simpler approach: package-level functions

go
```
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Printf("server: %s /\n", r.Method)
})

http.ListenAndServe(fmt.Sprintf(":%d", serverPort), nil)
```

This uses Go's default HTTP multiplexer, available as `http.DefaultServeMux`.

- `http.HandleFunc()` registers the route on `http.DefaultServeMux`.
- `http.ListenAndServe(addr, nil)` uses `http.DefaultServeMux` because the handler argument is `nil`.

Conceptually, this is equivalent to:

go
```
http.ListenAndServe(
    fmt.Sprintf(":%d", serverPort),
    http.DefaultServeMux,
)
```

The package-level functions are convenient for small applications, quick prototypes, and simple servers.

## 3. Key differences

| Feature | Custom mux + `http.Server` | Package-level functions |
| --- | --- | --- |
| Router | Explicit custom mux | Global `http.DefaultServeMux` |
| Route registration | `mux.HandleFunc()` | `http.HandleFunc()` |
| Server configuration | Full control through `http.Server` | Limited through `ListenAndServe()` |
| Multiple independent servers | Easy to configure separately | Possible, but routing is shared if using the default mux |
| Testability and isolation | Easier to isolate routes | Global state can complicate tests |
| Timeouts and other settings | Easy to configure | Requires constructing `http.Server` |
| Best suited for | Larger apps and production services | Small apps and prototypes |

## 4. Why the custom mux is often preferable

Suppose your application has two HTTP servers: one for your public API and another for internal metrics.

With custom muxes, you can keep their routes separate:

go
```
apiMux := http.NewServeMux()
apiMux.HandleFunc("/users", usersHandler)

metricsMux := http.NewServeMux()
metricsMux.HandleFunc("/metrics", metricsHandler)

apiServer := &http.Server{
    Addr:    ":8080",
    Handler: apiMux,
}

metricsServer := &http.Server{
    Addr:    ":9090",
    Handler: metricsMux,
}
```

Each server handles only the routes registered on its own mux. You can also configure different timeouts and other server settings.

With `http.HandleFunc()`, routes are registered globally on `http.DefaultServeMux`, so separate servers using `nil` as their handler share that router.

## 5. One important difference: error handling

Your first version explicitly checks the error returned by `ListenAndServe()`:

go
```
if err := server.ListenAndServe(); err != nil {
    if !errors.Is(err, http.ErrServerClosed) {
        fmt.Printf("error running http server: %s\n", err)
    }
}
```

The second version:

go
```
http.ListenAndServe(":8080", nil)
```

still returns an error, but ignores it if you don't check the return value.

A better minimal version is:

go
```
if err := http.ListenAndServe(":8080", nil); err != nil {
    log.Printf("HTTP server error: %v", err)
}
```

Note that `http.ErrServerClosed` is relevant when you explicitly shut down or close a server. A plain `ListenAndServe()` normally returns a non-nil error when it cannot start, such as when the port is already in use.

## Which one should you use?

I'd recommend `http.NewServeMux()` with an explicit `http.Server` for most real applications. It makes routing dependencies explicit, avoids relying on global mutable state, and gives you control over timeouts and graceful shutdown.

Use the package-level functions when you want the simplest possible server and don't need those additional controls.

One final detail: in both of your examples, the handler only logs the HTTP method. It doesn't write a response body or set a status explicitly, so Go will normally return an empty `200 OK` response.