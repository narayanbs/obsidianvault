
By the end of this tutorial, you’ll understand:

- **Basic HTTP Operations**: How to make GET and POST requests using Go’s `net/http` package
- **Request Customization**: Adding headers, timeouts, and custom HTTP clients for production use
- **Error Handling**: Proper [error handling](https://www.digitalocean.com/community/tutorials/handling-errors-in-go) patterns for HTTP requests and responses
- **JSON Processing**: Parsing JSON responses and sending JSON data in requests
- **AI Integration**: Modern patterns for integrating with AI APIs and microservices
- **Performance Optimization**: Connection pooling, timeouts, and retry mechanisms
- **Production Best Practices**: Security considerations, logging, and monitoring

## Making a GET Request

The Go [`net/http`](https://pkg.go.dev/net/http) package has a few different ways to use it as a client. You can use a common, global HTTP client with functions such as [`http.Get`](https://pkg.go.dev/net/http#Get) to quickly make an HTTP `GET` request with only a URL and a body, or you can create an [`http.Request`](https://pkg.go.dev/net/http#Request) to begin customizing certain aspects of the individual request. In this section, you will create an initial program using `http.Get` to make an HTTP request, and then you will update it to use an `http.Request` with the default HTTP client.

### Using http.Get to Make a Request


First, make the `projects` directory and navigate to it:

```bash
mkdir projects
cd projects
```

Next, make the directory for your project and navigate to it. In this case, use the directory `httpclient`:

```bash
mkdir httpclient
cd httpclient
```

Inside the `httpclient` directory, use `nano`, or your favorite editor, to open the `main.go` file:

```bash
nano main.go
```

In the `main.go` file, begin by adding these lines:


```go
package main

import (
 "errors"
 "fmt"
 "net/http"
 "os"
 "time"
)

const serverPort = 3333
```


Next, create a `main` function in the `main.go` file and set up a goroutine to start an HTTP server:


```go
...
func main() {
 go func() {
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
 }()

 time.Sleep(100 * time.Millisecond)
```

Your HTTP server is set up to use `fmt.Printf` to print information about incoming requests whenever the root `/` path is requested. It’s also set to listen on `serverPort`. Finally, once you start up the server goroutine, your program uses `time.Sleep` for a short amount of time. This sleep time allows the HTTP server the time it needs to start up and start serving responses to the request you’ll be making next.

Now, also in the `main` function, set up the request URL using `fmt.Sprintf` to combine the `http://localhost` hostname with the `serverPort` value the server is listening on. Then, use `http.Get` to make a request to that URL, as shown below:

main.go

```go
...
 requestURL := fmt.Sprintf("http://localhost:%d", serverPort)
 res, err := http.Get(requestURL)
 if err != nil {
  fmt.Printf("error making http request: %s\n", err)
  os.Exit(1)
 }

 fmt.Printf("client: got response!\n")
 fmt.Printf("client: status code: %d\n", res.StatusCode)
}
```

Save and close the file when you’re done.

To run your program, use the `go run` command and provide the `main.go` file to it:

```bash
go run main.go
```

You will see the following output:

```bash
Outputserver: GET /
client: got response!
client: status code: 200
```


The `http.Get` function is useful for quick HTTP requests like the one you made in this section. However, `http.Request` provides a broader range of options for customizing your request.

### Using http.Request to Make a Request

In contrast to `http.Get`, the `http.Request` function provides you with greater control over the request, other than just the HTTP method and the URL being requested. You won’t be using additional features yet, but by using an `http.Request` now, you’ll be able to add those customizations later in this tutorial.

In your code, the first update is to change the HTTP server handler to return a fake JSON data response using `fmt.Fprintf`. If this were a full HTTP server, this data would be generated using Go’s [`encoding/json`](https://pkg.go.dev/encoding/json) package. If you’d like to learn more about using JSON in Go, our [How To Use JSON in Go](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go) tutorial is available. In addition, you will also need to include `io` package as an import for use later in this update.

Now, open your `main.go` file again and update your program to start using an `http.Request` as shown below:


```go
package main

import (
 ...
 "io"
 ...
)

...

func main() {
 ...
 mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  fmt.Printf("server: %s /\n", r.Method)
  fmt.Fprintf(w, \`{"message": "hello!"}\`)
 })
 ...
```

Now, update your HTTP request code so that instead of using `http.Get` to make a request to the server, you use `http.NewRequest` and `http.DefaultClient` ’s `Do` method:

main.go

```go
...
 requestURL := fmt.Sprintf("http://localhost:%d", serverPort)
 req, err := http.NewRequest(http.MethodGet, requestURL, nil)
 if err != nil {
  fmt.Printf("client: could not create request: %s\n", err)
  os.Exit(1)
 }

 res, err := http.DefaultClient.Do(req)
 if err != nil {
  fmt.Printf("client: error making http request: %s\n", err)
  os.Exit(1)
 }

 fmt.Printf("client: got response!\n")
 fmt.Printf("client: status code: %d\n", res.StatusCode)

 resBody, err := io.ReadAll(res.Body)
 if err != nil {
  fmt.Printf("client: could not read response body: %s\n", err)
  os.Exit(1)
 }
 fmt.Printf("client: response body: %s\n", resBody)
}
```

In this update, you use the `http.NewRequest` function to generate an `http.Request` value, or handle the error if the value can’t be created. Unlike the `http.Get` function, though, the `http.NewRequest` function doesn’t send an HTTP request to the server right away. Since it doesn’t send the request right away, you can make any changes you’d like to the request before it’s sent.

Once the `http.Request` is created and configured, you use the `Do` method of `http.DefaultClient` to send the request to the server. The `http.DefaultClient` value is Go’s default HTTP client, the same you’ve been using with `http.Get`. This time, though, you’re using it directly to tell it to send your `http.Request`. The `Do` method of the HTTP client returns the same values you received from the `http.Get` function so that you can handle the response in the same way.

After you’ve printed the request results, you use the [`io.ReadAll`](https://pkg.go.dev/io#ReadAll) function to read the HTTP response’s `Body`. The `Body` is an [`io.ReadCloser`](https://pkg.go.dev/io#ReadCloser) value, a combination of [`io.Reader`](https://pkg.go.dev/io#Reader) and [`io.Closer`](https://pkg.go.dev/io#Closer), which means you can read the body’s data using anything that can read from an [`io.Reader`](https://pkg.go.dev/io#Reader) value. The `io.ReadAll` function is useful because it will read from an `io.Reader` until it either gets to the end of the data or encounters an `error`. Then it will either return the data as a `[]byte` value you can print using `fmt.Printf`, or the `error` value it encountered.

To run your updated program, save your changes and use the `go run` command:

```bash
go run main.go
```

This time, your output should look very similar to before, but with one addition:

```bash
Outputserver: GET /
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}
```


In this section, you created a program with an HTTP server that you made HTTP requests to in various ways. First, you used the `http.Get` function to make a `GET` request to the server using only the server’s URL. Then, you updated your program to use `http.NewRequest` to create an `http.Request` value. Once that was created, you used the `Do` method of Go’s default HTTP client, `http.DefaultClient`, to make the request and print the `http.Response` `Body` to the output.

### Advanced HTTP Client Configuration

For production applications, you’ll often need more sophisticated HTTP client configurations. Here’s an example of a production-ready HTTP client with connection pooling, timeouts, and retry logic:

```go
// Production HTTP client with advanced configuration
func createProductionClient() *http.Client {
    transport := &http.Transport{
        MaxIdleConns:        100,              // Maximum idle connections
        MaxIdleConnsPerHost: 10,               // Maximum idle connections per host
        IdleConnTimeout:     90 * time.Second, // Idle connection timeout
        DisableCompression:  false,            // Enable compression
        DisableKeepAlives:   false,            // Enable keep-alives
    }

    return &http.Client{
        Transport: transport,
        Timeout:   30 * time.Second,
        CheckRedirect: func(req *http.Request, via []*http.Request) error {
            // Custom redirect handling
            if len(via) >= 10 {
                return errors.New("stopped after 10 redirects")
            }
            return nil
        },
    }
}
```

### AI Integration Patterns

Modern applications often need to integrate with [AI services](https://www.digitalocean.com/products/gradient/platform). Here’s how you can structure HTTP requests for AI APIs:

```go
// AI API integration example
type AIRequest struct {
    Model    string \`json:"model"\`
    Messages []struct {
        Role    string \`json:"role"\`
        Content string \`json:"content"\`
    } \`json:"messages"\`
    MaxTokens int \`json:"max_tokens"\`
}

func callAIService(client *http.Client, apiKey string, prompt string) (*http.Response, error) {
    aiReq := AIRequest{
        Model: "gpt-3.5-turbo",
        Messages: []struct {
            Role    string \`json:"role"\`
            Content string \`json:"content"\`
        }{
            {Role: "user", Content: prompt},
        },
        MaxTokens: 150,
    }

    jsonData, err := json.Marshal(aiReq)
    if err != nil {
        return nil, err
    }

    req, err := http.NewRequest("POST", "https://api.openai.com/v1/chat/completions", bytes.NewBuffer(jsonData))
    if err != nil {
        return nil, err
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)
    req.Header.Set("User-Agent", "Go-HTTP-Client/1.0")

    return client.Do(req)
}
```

The [HTTP protocol](https://www.digitalocean.com/community/tutorials/http-1-1-vs-http-2-what-s-the-difference) uses more than just `GET` requests to communicate between programs, though. A `GET` request is useful when you want to receive information from the other program, but another HTTP method, the `POST` method, can be used when you want to send information from your program to the server.

## Sending a POST Request

In a [REST API](https://en.wikipedia.org/wiki/Representational_state_transfer), a `GET` request is only used for retrieving information from the server, so for your program to fully participate in a REST API, your program also needs to support sending `POST` requests. A `POST` request is almost the inverse of a `GET` request, where the client sends data to the server in the request’s body.

In this section, you will update your program to send your request as a `POST` request instead of a `GET` request. Your `POST` request will include a request body, and you will update your server to print out more information about the requests you’re making from the client.

To start making these updates, open your `main.go` file and add a few new packages that you’ll be using to your `import` statement:

main.go

```go
...

import (
 "bytes"
 "errors"
 "fmt"
 "io"
 "net/http"
 "os"
 "strings"
 "time"
)

...
```

Then, update your server handler function to print out various information about the request coming in, such as query string values, header values, and the request body:


```go
...
  mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
   fmt.Printf("server: %s /\n", r.Method)
   fmt.Printf("server: query id: %s\n", r.URL.Query().Get("id"))
   fmt.Printf("server: content-type: %s\n", r.Header.Get("content-type"))
   fmt.Printf("server: headers:\n")
   for headerName, headerValue := range r.Header {
    fmt.Printf("\t%s = %s\n", headerName, strings.Join(headerValue, ", "))
   }

   reqBody, err := io.ReadAll(r.Body)
   if err != nil {
    fmt.Printf("server: could not read request body: %s\n", err)
   }
   fmt.Printf("server: request body: %s\n", reqBody)

   fmt.Fprintf(w, \`{"message": "hello!"}\`)
  })
...
```

In this update to the server’s HTTP request handler, you add a few more helpful `fmt.Printf` statements to see information about the request coming in. You use `r.URL.Query().Get` to get a query string value named `id`, and `r.Header.Get` to get the value of a header called `content-type`. You also use a `for` loop with `r.Header` to print the name and value of each HTTP header the server received. This information can be useful for troubleshooting issues if your client or server isn’t acting the way you expect. Finally, you also used the `io.ReadAll` function to read the HTTP request’s body in `r.Body`.

After updating the server handler function, update the `main` function’s request code so that it’s sending a `POST` request with a request body:


```go
...
 time.Sleep(100 * time.Millisecond)
 
 jsonBody := []byte(\`{"client_message": "hello, server!"}\`)
 bodyReader := bytes.NewReader(jsonBody)

 requestURL := fmt.Sprintf("http://localhost:%d?id=1234", serverPort)
 req, err := http.NewRequest(http.MethodPost, requestURL, bodyReader)
...
```

In your update to the `main` function’s request, one of the new values you’re defining is the `jsonBody` value. In this example, the value is represented as a `[]byte` instead of the standard `string` because if you use the `encoding/json` package to encode JSON data, it will give you a `[]byte` back instead of a `string`.

The next value, the `bodyReader`, is a [`bytes.Reader`](https://pkg.go.dev/bytes#Reader) that wraps the `jsonBody` data. An `http.Request` body requires the value to be an `io.Reader`, and `jsonBody` ’s `[]byte` value doesn’t implement `io.Reader`, so you wouldn’t be able to use it as a request body on its own. The `bytes.Reader` value exists to provide that `io.Reader` interface, so you can use the `jsonBody` value as the request body.

The `requestURL` value is also updated to include an `id=1234` query string value, primarily to show how a query string value can also be included in the request URL along with other standard URL components.

Finally, the `http.NewRequest` function call is updated to use a `POST` method with `http.MethodPost`, and the request body is included by updating the last parameter from a `nil` body to `bodyReader`, the JSON data `io.Reader`.

Once you’ve saved your changes, you can use `go run` to run your program:

```bash
go run main.go
```

The output will be longer than before because of your updates to the server to show additional information:

```bash
Outputserver: POST /
server: query id: 1234
server: content-type: 
server: headers:
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
server: request body: {"client_message": "hello, server!"}
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}
```


### Advanced Error Handling and Retry Logic

Production applications need robust error handling. Here’s an enhanced version with retry logic and comprehensive error handling:

```go
// Enhanced error handling with retry logic
func makeRequestWithRetry(client *http.Client, req *http.Request, maxRetries int) (*http.Response, error) {
    var lastErr error
    
    for attempt := 0; attempt <= maxRetries; attempt++ {
        resp, err := client.Do(req)
        if err == nil {
            // Check for HTTP error status codes
            if resp.StatusCode >= 400 {
                resp.Body.Close()
                if attempt < maxRetries && isRetryableError(resp.StatusCode) {
                    time.Sleep(time.Duration(attempt+1) * time.Second)
                    continue
                }
                return nil, fmt.Errorf("HTTP error: %d %s", resp.StatusCode, resp.Status)
            }
            return resp, nil
        }
        
        lastErr = err
        
        // Check if error is retryable
        if attempt < maxRetries && isRetryableNetworkError(err) {
            time.Sleep(time.Duration(attempt+1) * time.Second)
            continue
        }
        
        break
    }
    
    return nil, fmt.Errorf("request failed after %d attempts: %w", maxRetries+1, lastErr)
}

func isRetryableError(statusCode int) bool {
    return statusCode == 429 || statusCode == 502 || statusCode == 503 || statusCode == 504
}

func isRetryableNetworkError(err error) bool {
    if netErr, ok := err.(net.Error); ok {
        return netErr.Temporary() || netErr.Timeout()
    }
    return false
}
```

### JSON Processing Best Practices

When working with JSON in HTTP requests, follow these patterns:

```go
// Structured JSON request/response handling
type User struct {
    ID    int    \`json:"id"\`
    Name  string \`json:"name"\`
    Email string \`json:"email"\`
}

type APIResponse struct {
    Success bool   \`json:"success"\`
    Message string \`json:"message"\`
    Data    User   \`json:"data,omitempty"\`
}

func createUser(client *http.Client, user User) (*APIResponse, error) {
    jsonData, err := json.Marshal(user)
    if err != nil {
        return nil, fmt.Errorf("failed to marshal JSON: %w", err)
    }

    req, err := http.NewRequest("POST", "https://api.example.com/users", bytes.NewBuffer(jsonData))
    if err != nil {
        return nil, fmt.Errorf("failed to create request: %w", err)
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Accept", "application/json")

    resp, err := client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("request failed: %w", err)
    }
    defer resp.Body.Close()

    var apiResp APIResponse
    if err := json.NewDecoder(resp.Body).Decode(&apiResp); err != nil {
        return nil, fmt.Errorf("failed to decode response: %w", err)
    }

    return &apiResp, nil
}
```

Typically in an HTTP request, the client or the server will tell the other the type of content it’s sending in the body. As you saw in the last output, though, your HTTP request didn’t include a `Content-Type` header to tell the server how to interpret the body’s data. In the next section, you’ll make a few updates to customize your HTTP request, including setting a `Content-Type` header to let the server know the type of data you’re sending.

## Customizing an HTTP Request

Over time, [HTTP requests and responses](https://www.digitalocean.com/community/tutorials/how-to-troubleshoot-common-http-error-codes) have been used to send a greater variety of data between clients and servers. At one point, HTTP clients could assume the data they’re receiving from an HTTP server is HTML and have a good chance of being correct. Now, though, it could be HTML, JSON, music, video, or any number of other data types. To provide more information about the data being sent over HTTP, the protocol includes HTTP headers, and one of those important headers is the `Content-Type` header. This header tells the server (or client, depending on the direction of the data) how to interpret the data it’s receiving.

In this section, you will update your program to set the `Content-Type` header on your HTTP request so the server knows it’s receiving JSON data. You will also update your program to use an HTTP client other than Go’s default `http.DefaultClient` so that you can customize how the request is sent.

To make these updates, open your `main.go` file again and update your `main` function like so:

main.go

```go
...

  req, err := http.NewRequest(http.MethodPost, requestURL, bodyReader)
  if err != nil {
   fmt.Printf("client: could not create request: %s\n", err)
   os.Exit(1)
  }
  req.Header.Set("Content-Type", "application/json")

  client := http.Client{
  Timeout: 30 * time.Second,
  }

  res, err := client.Do(req)
  if err != nil {
   fmt.Printf("client: error making http request: %s\n", err)
   os.Exit(1)
  }

...
```

In this update, you access the `http.Request` headers using `req.Header`, and then set the value of the `Content-Type` header on the request to `application/json`. The `application/json` media type is defined in the list of [media types](https://www.iana.org/assignments/media-types/media-types.xhtml) as the media type for JSON. This way, when the server receives your request, it knows to interpret the body as JSON and not, for example, [XML](https://en.wikipedia.org/wiki/XML).

The next update is to create your own `http.Client` instance in the `client` variable. In this client, you set the `Timeout` value to 30 seconds. This is important because it says that any requests made with the client will give up and stop trying to receive a response after 30 seconds. Go’s default `http.DefaultClient` doesn’t specify a timeout, so if you make a request using that client, it will wait until it receives a response, is disconnected by the server, or your program ends. If you have many requests hanging around like this waiting for a response, you could be using a large number of resources on your computer. Setting a `Timeout` value limits how long a request will wait by the time you define.

Finally, you updated your request to use the `Do` method of your `client` variable. You don’t need to make any other changes here because you’ve been calling `Do` on an `http.Client` value the whole time. Go’s default HTTP client, `http.DefaultClient`, is just an `http.Client` that’s created by default. So, when you called `http.Get`, the function was calling the `Do` method for you, and when you updated your request to use `http.DefaultClient`, you were using that `http.Client` directly. The only difference now is that you created the `http.Client` value you’re using this time.

Now, save your file and run your program using `go run`:

```bash
go run main.go
```

Your output should be very similar to the previous output but with more information about the content type:

```bash
Outputserver: POST /
server: query id: 1234
server: content-type: application/json
server: headers:
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
        Content-Type = application/json
server: request body: {"client_message": "hello, server!"}
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}
```

You’ll see there’s a value from the server for `content-type`, and there’s a `Content-Type` header being sent by the client. This is how you could have the same HTTP request path serving both a JSON and an XML API at the same time. By specifying the request’s content type, the server and the client can interpret the data differently.

This example doesn’t trigger the client timeout you configured, though. To see what happens when a request takes too long and the timeout is triggered, open your `main.go` file and add a `time.Sleep` function call to your HTTP server handler function. Then, make the `time.Sleep` last for longer than the timeout you specified. In this case, you’ll set it for 35 seconds:

main.go

```go
...

func main() {
 go func() {
  mux := http.NewServeMux()
  mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
   ... 
   fmt.Fprintf(w, \`{"message": "hello!"}\`)
   time.Sleep(35 * time.Second)
  })
  ...
 }()
 ...
}
```

Now, save your changes and run your program using `go run`:

```bash
go run main.go
```

When you run it this time, it will take longer to exit than before because it won’t exit until after the HTTP request is finished. Since you added the `time.Sleep(35 * time.Second)`, the HTTP request won’t complete until the 30-second timeout is reached:

```bash
Outputserver: POST /
server: query id: 1234
server: content-type: application/json
server: headers:
        Content-Type = application/json
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
server: request body: {"client_message": "hello, server!"}
client: error making http request: Post "http://localhost:3333?id=1234": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
exit status 1
```

In this program output, you see the server received the request and processed it, but when it reached the end of the HTTP handler function where your `time.Sleep` function call is, it started sleeping for 35 seconds. At the same time, the timeout for your HTTP request is counting down and reaches the limit of 30 seconds before the HTTP request finishes. This results in the `client.Do` method call failing with a `context deadline exceeded` error because the request’s 30-second deadline passed. Then, your program exits with a failure status code of `1` using `os.Exit(1)`.

In this section, you updated your program to customize an HTTP request by adding a `Content-Type` header to it. You also updated your program to create a new `http.Client` with a 30-second timeout, and then used that client to make an HTTP request. You also tested the 30-second timeout by adding a `time.Sleep` to your HTTP request handler. Finally, you also saw why it’s important to use your own `http.Client` values with timeouts set if you want to avoid many requests potentially idling forever.

## Advanced Patterns: Microservices and AI Integration in Go

### Microservices Communication Patterns

In modern [microservices architectures](https://www.digitalocean.com/solutions/microservices-hosting), HTTP clients often need to communicate with multiple services. Here’s how to structure this:

```go
// Service discovery and load balancing
type ServiceClient struct {
    BaseURL    string
    HTTPClient *http.Client
    CircuitBreaker *CircuitBreaker
}

func (sc *ServiceClient) CallService(endpoint string, payload interface{}) (*http.Response, error) {
    if sc.CircuitBreaker.IsOpen() {
        return nil, errors.New("circuit breaker is open")
    }

    jsonData, err := json.Marshal(payload)
    if err != nil {
        return nil, err
    }

    req, err := http.NewRequest("POST", sc.BaseURL+endpoint, bytes.NewBuffer(jsonData))
    if err != nil {
        return nil, err
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("X-Service-Name", "user-service")

    resp, err := sc.HTTPClient.Do(req)
    if err != nil {
        sc.CircuitBreaker.RecordFailure()
        return nil, err
    }

    sc.CircuitBreaker.RecordSuccess()
    return resp, nil
}
```

This Go code block defines a `ServiceClient` struct and a method `CallService` designed for making robust HTTP requests to other microservices.

**`ServiceClient` Struct:** This struct acts as a blueprint for connecting to a specific microservice. It holds:

- `BaseURL`: The base address of the microservice (e.g., `http://api.userservice.com`).
- `HTTPClient`: An instance of Go’s standard `http.Client`, which is typically configured with important settings like timeouts to prevent requests from hanging indefinitely.
- `CircuitBreaker`: An implementation of the circuit breaker pattern. This is a crucial component for fault tolerance in microservices. It monitors the health of the remote service and can “open” (preventing further requests) if the service is failing, protecting both the calling service and the failing service from being overwhelmed.

**`CallService` Method:** This method is responsible for actually making the HTTP call to the microservice. Here’s a step-by-step breakdown:

1. **Circuit Breaker Check:** Before attempting any network request, it first checks if the `CircuitBreaker` is “open.” If it is, it means the target microservice is likely experiencing issues, so `CallService` immediately returns an error without even trying to send a request. This prevents unnecessary network traffic and protects the failing service.
2. **Prepare Request Payload:** The `payload` (which can be any Go data structure) is converted into a JSON format using `json.Marshal`. This is a common way to send structured data between services.
3. **Create HTTP Request:** An `http.NewRequest` is created.
	- It’s set as a `POST` request, indicating data will be sent to the server.
		- The full URL is constructed by combining the `ServiceClient` ’s `BaseURL` with the specific `endpoint` for the desired operation (e.g., `/users/create`).
		- The JSON-encoded `payload` is attached as the request body.
4. **Set HTTP Headers:**
	- `Content-Type: application/json`: This header tells the receiving service that the request body contains JSON data.
		- `X-Service-Name: user-service`: This is a custom header that can be used for tracing, logging, or routing purposes within a microservices architecture, identifying which service is making the call.
5. **Execute the Request:** The `sc.HTTPClient.Do(req)` call sends the prepared HTTP request over the network. This is where the actual communication happens.
6. **Update Circuit Breaker State:**
	- If an error occurs during the `HTTPClient.Do` call (e.g., network issues, timeout), the `sc.CircuitBreaker.RecordFailure()` method is called. This informs the circuit breaker that a request failed, potentially leading it to “open” if enough failures occur.
		- If the request is successful (even if the server returns an error status code like 404 or 500, as long as the network call itself completes), `sc.CircuitBreaker.RecordSuccess()` is called, indicating a successful interaction.
7. **Return Result:** Finally, the method returns the `http.Response` received from the microservice and any error that occurred during the process.

### AI API Integration Patterns

Modern applications increasingly integrate with [AI services](https://www.digitalocean.com/products/gradient/platform). Here are patterns for common AI integrations:

```go
// OpenAI API integration
type OpenAIClient struct {
    APIKey     string
    BaseURL    string
    HTTPClient *http.Client
}

func (c *OpenAIClient) GenerateText(prompt string) (string, error) {
    requestBody := map[string]interface{}{
        "model": "gpt-3.5-turbo",
        "messages": []map[string]string{
            {"role": "user", "content": prompt},
        },
        "max_tokens": 150,
    }

    jsonData, err := json.Marshal(requestBody)
    if err != nil {
        return "", err
    }

    req, err := http.NewRequest("POST", c.BaseURL+"/v1/chat/completions", bytes.NewBuffer(jsonData))
    if err != nil {
        return "", err
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+c.APIKey)

    resp, err := c.HTTPClient.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()

    var response struct {
        Choices []struct {
            Message struct {
                Content string \`json:"content"\`
            } \`json:"message"\`
        } \`json:"choices"\`
    }

    if err := json.NewDecoder(resp.Body).Decode(&response); err != nil {
        return "", err
    }

    if len(response.Choices) == 0 {
        return "", errors.New("no response from AI service")
    }

    return response.Choices[0].Message.Content, nil
}

// Anthropic Claude API integration
func (c *OpenAIClient) CallClaude(prompt string) (string, error) {
    requestBody := map[string]interface{}{
        "model": "claude-3-sonnet-20240229",
        "max_tokens": 1000,
        "messages": []map[string]string{
            {"role": "user", "content": prompt},
        },
    }

    jsonData, err := json.Marshal(requestBody)
    if err != nil {
        return "", err
    }

    req, err := http.NewRequest("POST", "https://api.anthropic.com/v1/messages", bytes.NewBuffer(jsonData))
    if err != nil {
        return "", err
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("x-api-key", c.APIKey)
    req.Header.Set("anthropic-version", "2023-06-01")

    resp, err := c.HTTPClient.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()

    var response struct {
        Content []struct {
            Text string \`json:"text"\`
        } \`json:"content"\`
    }

    if err := json.NewDecoder(resp.Body).Decode(&response); err != nil {
        return "", err
    }

    if len(response.Content) == 0 {
        return "", errors.New("no response from Claude")
    }

    return response.Content[0].Text, nil
}
```

This `CallClaude` function demonstrates how to interact with the Anthropic Claude API to generate text. It takes a `prompt` string as input and returns the generated text.

Here’s a breakdown of its functionality:

1. **Request Body Construction**: It prepares a JSON request body containing the desired Claude model (`claude-3-sonnet-20240229`), `max_tokens` for the response length, and the user’s `prompt` formatted as a message.
2. **HTTP Request Creation**: An `http.NewRequest` is used to create a `POST` request to the Anthropic API endpoint (`https://api.anthropic.com/v1/messages`). The marshaled JSON body is included in the request.
3. **Header Configuration**: Essential headers are set: `Content-Type` to `application/json`, `x-api-key` for authentication using the client’s API key, and `anthropic-version` to specify the API version.
4. **Request Execution**: The `c.HTTPClient.Do(req)` method sends the request, and `defer resp.Body.Close()` ensures the response body is closed after processing.
5. **Response Parsing**: The function defines a struct to unmarshal the JSON response, specifically extracting the `text` content from the `content` array. It then decodes the response body into this struct.
6. **Error Handling**: It checks for errors during JSON marshaling, request creation, HTTP execution, and JSON decoding. It also returns an error if the Claude API returns no content.

### Streaming Responses for AI APIs

For AI APIs that support streaming, handle responses efficiently:

```go
func (c *OpenAIClient) StreamGenerateText(prompt string, onChunk func(string)) error {
    requestBody := map[string]interface{}{
        "model": "gpt-3.5-turbo",
        "messages": []map[string]string{
            {"role": "user", "content": prompt},
        },
        "stream": true,
    }

    jsonData, err := json.Marshal(requestBody)
    if err != nil {
        return err
    }

    req, err := http.NewRequest("POST", c.BaseURL+"/v1/chat/completions", bytes.NewBuffer(jsonData))
    if err != nil {
        return err
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+c.APIKey)

    resp, err := c.HTTPClient.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    scanner := bufio.NewScanner(resp.Body)
    for scanner.Scan() {
        line := scanner.Text()
        if strings.HasPrefix(line, "data: ") {
            data := strings.TrimPrefix(line, "data: ")
            if data == "[DONE]" {
                break
            }

            var chunk struct {
                Choices []struct {
                    Delta struct {
                        Content string \`json:"content"\`
                    } \`json:"delta"\`
                } \`json:"choices"\`
            }

            if err := json.Unmarshal([]byte(data), &chunk); err == nil {
                if len(chunk.Choices) > 0 && chunk.Choices[0].Delta.Content != "" {
                    onChunk(chunk.Choices[0].Delta.Content)
                }
            }
        }
    }

    return scanner.Err()
}
```

## Best Practices for Production HTTP Clients

When building HTTP clients for production in Go, follow these best practices to ensure reliability, security, and maintainability.

| Best Practice | Description | Example / Tip |
| --- | --- | --- |
| Use Timeouts | Always set timeouts to avoid hanging requests. | `client.Timeout = 30 * time.Second` |
| Secure TLS Configuration | Enforce minimum TLS version and validate certificates. | See `tls.Config{MinVersion: tls.VersionTLS12}` |
| Connection Pooling | Reuse connections for efficiency and performance. | Use `http.Transport` with `MaxIdleConns` and `IdleConnTimeout` |
| Handle Errors Robustly | Check for and handle all errors, including non-2xx responses. | `if resp.StatusCode != http.StatusOK { ... }` |
| Limit Redirects | Prevent infinite redirect loops. | `client.CheckRedirect = ...` |
| Set Appropriate Headers | Always set required headers (e.g., `Content-Type`, authentication). | `req.Header.Set("Authorization", "Bearer ...")` |
| Log Requests and Responses | Add logging for observability and debugging. | Wrap transport with a logging middleware |
| Monitor and Instrument | Collect metrics for latency, errors, and throughput. | Integrate with Prometheus or similar tools |
| Use Contexts | Pass `context.Context` to support cancellation and deadlines. | `req.WithContext(ctx)` |
| Protect Sensitive Data | Never log secrets or sensitive payloads. | Mask or omit sensitive fields in logs |
| Retry with Backoff | Implement retries with exponential backoff for transient errors. | Use libraries like `github.com/cenkalti/backoff` |
| Respect Rate Limits | Handle HTTP 429 and back off as needed. | Check for `429 Too Many Requests` and use `Retry-After` header |

These practices help ensure your HTTP clients are robust, secure, and production-ready.

### 1\. Security Considerations

When building production HTTP clients, consider these security best practices:

```go
// Secure HTTP client configuration
func createSecureClient() *http.Client {
    transport := &http.Transport{
        TLSClientConfig: &tls.Config{
            MinVersion: tls.VersionTLS12, // Minimum TLS 1.2
        },
        DisableCompression: false,
        MaxIdleConns:       100,
        IdleConnTimeout:    90 * time.Second,
    }

    return &http.Client{
        Transport: transport,
        Timeout:   30 * time.Second,
    }
}

// Request signing for API authentication
func signRequest(req *http.Request, secret string) {
    timestamp := time.Now().Unix()
    req.Header.Set("X-Timestamp", strconv.FormatInt(timestamp, 10))
    
    // Create signature (simplified example)
    signature := createSignature(req.Method, req.URL.Path, timestamp, secret)
    req.Header.Set("X-Signature", signature)
}
```

### 2\. Monitoring and Observability

Add logging and metrics to your HTTP clients:

```go
// HTTP client with logging and metrics
type LoggingTransport struct {
    Transport http.RoundTripper
    Logger    *log.Logger
}

func (t *LoggingTransport) RoundTrip(req *http.Request) (*http.Response, error) {
    start := time.Now()
    
    t.Logger.Printf("HTTP %s %s", req.Method, req.URL.String())
    
    resp, err := t.Transport.RoundTrip(req)
    
    duration := time.Since(start)
    if err != nil {
        t.Logger.Printf("HTTP %s %s failed after %v: %v", req.Method, req.URL.String(), duration, err)
    } else {
        t.Logger.Printf("HTTP %s %s completed in %v with status %d", req.Method, req.URL.String(), duration, resp.StatusCode)
    }
    
    return resp, err
}
```

## Frequently Asked Questions

### 1\. How do I make a GET request in Go?

The simplest way to make a GET request in Go is using the `http.Get` function:

```go
resp, err := http.Get("https://api.example.com/data")
if err != nil {
    log.Fatal(err)
}
defer resp.Body.Close()

body, err := io.ReadAll(resp.Body)
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(body))
```

For more control, use `http.NewRequest` with a custom client:

```go
req, err := http.NewRequest("GET", "https://api.example.com/data", nil)
if err != nil {
    log.Fatal(err)
}

client := &http.Client{Timeout: 10 * time.Second}
resp, err := client.Do(req)
if err != nil {
    log.Fatal(err)
}
defer resp.Body.Close()
```

### 2\. How can I send JSON data in a POST request in Go?

To send JSON data in a POST request, marshal your data and set the appropriate headers:

```go
type User struct {
    Name  string \`json:"name"\`
    Email string \`json:"email"\`
}

user := User{Name: "John Doe", Email: "john@example.com"}
jsonData, err := json.Marshal(user)
if err != nil {
    log.Fatal(err)
}

req, err := http.NewRequest("POST", "https://api.example.com/users", bytes.NewBuffer(jsonData))
if err != nil {
    log.Fatal(err)
}

req.Header.Set("Content-Type", "application/json")

client := &http.Client{}
resp, err := client.Do(req)
if err != nil {
    log.Fatal(err)
}
defer resp.Body.Close()
```

### 3\. How do I add headers to an HTTP request in Go?

You can add headers using the `Header.Set` method on the request:

```go
req, err := http.NewRequest("GET", "https://api.example.com/data", nil)
if err != nil {
    log.Fatal(err)
}

// Add custom headers
req.Header.Set("Authorization", "Bearer your-token")
req.Header.Set("User-Agent", "MyApp/1.0")
req.Header.Set("Accept", "application/json")
req.Header.Set("X-Custom-Header", "custom-value")

client := &http.Client{}
resp, err := client.Do(req)
```

### 4\. What is the best way to parse JSON responses in Go?

Use the `encoding/json` package with structured types for type-safe JSON parsing:

```go
type APIResponse struct {
    Status  string \`json:"status"\`
    Message string \`json:"message"\`
    Data    struct {
        ID    int    \`json:"id"\`
        Name  string \`json:"name"\`
    } \`json:"data"\`
}

resp, err := http.Get("https://api.example.com/data")
if err != nil {
    log.Fatal(err)
}
defer resp.Body.Close()

var apiResp APIResponse
if err := json.NewDecoder(resp.Body).Decode(&apiResp); err != nil {
    log.Fatal(err)
}

fmt.Printf("Status: %s, Name: %s\n", apiResp.Status, apiResp.Data.Name)
```

For dynamic JSON, use `map[string]interface{}`:

```go
var result map[string]interface{}
if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
    log.Fatal(err)
}
```

### 5\. How can I set a timeout for HTTP requests in Go?

Set timeouts using the `Timeout` field on your HTTP client:

```go
// Global timeout for the entire request
client := &http.Client{
    Timeout: 30 * time.Second,
}

// Or use context for more granular control
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

req, err := http.NewRequestWithContext(ctx, "GET", "https://api.example.com/data", nil)
if err != nil {
    log.Fatal(err)
}

resp, err := client.Do(req)
if err != nil {
    if errors.Is(err, context.DeadlineExceeded) {
        log.Println("Request timed out")
    } else {
        log.Fatal(err)
    }
}
```

### 6\. When should I use Resty instead of net/http?

Use Resty when you need:

- **Simplified API**: Resty provides a more fluent, chainable API
- **Built-in JSON handling**: Automatic marshaling/unmarshaling
- **Request/Response middleware**: Built-in logging, retry, and error handling
- **Less boilerplate**: Reduces repetitive code

```go
// With net/http (more verbose)
req, err := http.NewRequest("POST", "https://api.example.com/users", bytes.NewBuffer(jsonData))
req.Header.Set("Content-Type", "application/json")
resp, err := client.Do(req)

// With Resty (more concise)
resp, err := resty.R().
    SetHeader("Content-Type", "application/json").
    SetBody(user).
    Post("https://api.example.com/users")
```

However, stick with `net/http` when you need:

- **Maximum control**: Fine-grained control over every aspect
- **Standard library**: No external dependencies
- **Performance**: Slightly better performance due to less abstraction
- **Learning**: Better understanding of HTTP fundamentals

## Conclusion

In this tutorial, you created a new program with an HTTP server and used Go’s `net/http` package to make HTTP requests to that server. First, you used the `http.Get` function to make a `GET` request to the server with Go’s default HTTP client. Then, you used `http.NewRequest` with `http.DefaultClient` ’s `Do` method to make a `GET` request. Next, you updated your request to make it a `POST` request with a body using `bytes.NewReader`. Finally, you used the `Set` method on an `http.Request` ’s `Header` field to set a request’s `Content-Type` header, and set a 30-second timeout on a request’s duration by creating your own HTTP client instead of using Go’s default client.

The [`net/http`](https://pkg.go.dev/net/http) package includes more than just the functionality you used in this tutorial. It also includes an [`http.Post`](https://pkg.go.dev/net/http#Post) function that can be used to make a `POST` request, similar to the `http.Get` function. The package also supports saving and retrieving [cookies](https://pkg.go.dev/net/http#CookieJar), among other features.

