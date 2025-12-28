

---


```c
#include <netinet/in.h>     // sockaddr_in, htons, AF_INET
#include <openssl/ssl.h>   // OpenSSL SSL/TLS functions and types
#include <stdio.h>         // printf, perror, FILE
#include <stdlib.h>        // EXIT_FAILURE
#include <string.h>        // memset, memcpy, strncmp
#include <sys/socket.h>    // socket, bind, listen, accept

int main(void) {

  // Create a TCP socket (IPv4, stream-based)
  int sockfd = socket(AF_INET, SOCK_STREAM, 0);

  // Define the server address structure
  struct sockaddr_in addr = {
      AF_INET,        // Address family: IPv4
      htons(8080),    // Port number (8080), converted to network byte order
      0               // IP address: 0 means INADDR_ANY (bind to all interfaces)
  };

  // Bind the socket to the address and port
  if (bind(sockfd, (struct sockaddr *)&addr, sizeof(addr)) < 0) {
    perror("bind");          // Print error if bind fails
    return EXIT_FAILURE;
  }

  // Mark the socket as passive, ready to accept incoming connections
  listen(sockfd, 10);        // Allow up to 10 queued connections

  // Accept one incoming client connection
  int clientfd = accept(sockfd, NULL, NULL);

  // Create an SSL context configured for a TLS server
  SSL_CTX *ctx = SSL_CTX_new(TLS_server_method());

  // Create a new SSL object for this client connection
  SSL *ssl = SSL_new(ctx);

  // Attach the SSL object to the accepted socket file descriptor
  SSL_set_fd(ssl, clientfd);

  // Load the server certificate chain from file
  SSL_use_certificate_chain_file(ssl, "fullchain");

  // Load the server private key from file (PEM format)
  SSL_use_PrivateKey_file(ssl, "thekey", SSL_FILETYPE_PEM);

  // Perform the TLS handshake with the client
  SSL_accept(ssl);

  // Buffer to hold incoming client data
  char buffer[1024] = {0};

  // Read encrypted data from the client over TLS
  SSL_read(ssl, buffer, 1023);

  // Skip the first 5 characters of the request ("GET /")
  // This assumes a request like: "GET /index.html HTTP/1.1"
  char *file_request = buffer + 5;

  // Buffer to store the HTTP response
  char response[1024] = {0};

  // Basic HTTP response headers
  char *metadata =
      "HTTP/1.0 200 OK\r\n"
      "Content-Type: text/html\r\n"
      "\r\n";

  // Copy HTTP headers into response buffer
  memcpy(response, metadata, strlen(metadata));

  // If the requested file is "index.html"
  if (strncmp(file_request, "index.html", 11) == 0) {

    // Open index.html for reading
    FILE *f = fopen("index.html", "r");

    // Read file contents into the response buffer after headers
    fread(response + strlen(metadata),
          1024 - strlen(metadata) - 1,
          1,
          f);

    fclose(f);
  } else {

    // If the requested file is not found
    char *error = "No page found";

    // Append error message to response body
    memcpy(response + strlen(metadata), error, strlen(error));
  }

  // Send the response to the client over TLS
  SSL_write(ssl, response, 1024);

  // Cleanly shut down the TLS connection
  SSL_shutdown(ssl);
}
```

---
