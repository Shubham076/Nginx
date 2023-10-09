### Nginnx Configuration

Certainly! NGINX is highly versatile and serves multiple purposes, from web servers to reverse proxies and beyond. Below are some of the most common use-cases for NGINX, combined with a brief configuration for each:

## 1. **Static Website Hosting**

To serve a simple static website:


```
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html;
}
```
- The `root` directive specifies the directory where NGINX should look for files to serve in response to client requests. It defines the root path of the content that will be delivered to the client. 
- And a client makes a request to `http://example.com/styles.css`, NGINX will attempt to serve the file located at `/var/www/html/styles.css` on the server's filesystem.

- Similarly, if a client makes a request to `http://example.com/images/photo.jpg`, NGINX will look for and serve the file at `/var/www/html/images/photo.jpg`.

- To summarize, the `root` directive in NGINX defines the base directory (or path) on the server's filesystem from which static files will be served in response to client requests.

## 2. **HTTPS Redirect**

Redirect all HTTP traffic to HTTPS:

```
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
``` 

## 3. **Reverse Proxy for a Web App**

To proxy requests to another server, commonly used for applications running on ports other than 80/443:


```
server {
    listen 80;
    server_name app.example.com;
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 4. **Load Balancing**

Balance traffic between multiple back-end servers, default **round-robin**

```upstream backend_servers {
    server backend1.example.com;
    server backend2.example.com;
    #... more servers if needed
}

server {
    listen 80;
    location / {
        proxy_pass http://backend_servers;
    }
}
```

## 5. **Rate Limiting**

Limit the rate at which clients can make requests:

```
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;

    server {
        listen 80;
        location / {
            limit_req zone=mylimit burst=5;
        }
    }
}
```

-   `limit_req_zone`: This directive sets up the rate limiting.
    
-   `$binary_remote_addr`: This variable is used to track the rate of requests based on the client's IP address. Using `$binary_remote_addr` instead of `$remote_addr` makes the storage slightly more efficient.
    
-   `zone=mylimit:10m`: This defines a shared memory zone named `mylimit` to store the state of incoming requests. `10m` allocates 10 megabytes of memory for this zone, which can track the rate for millions of IP addresses.
    
-   `rate=1r/s`: This sets the allowed request rate. Here, it's limited to one request per second.
-   Normally, clients are limited to one request per second.
-   If a client sends more than one request in less than a second but doesn't exceed the `burst` limit (5 requests), NGINX will queue the extra requests and process them without sending an error, as long as the burst isn't exceeded.

## 6. **Caching Proxy**

Use NGINX as a caching proxy to reduce load on backend servers:

```
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m;

server {
    listen 80;
    location / {
        proxy_cache my_cache;
        proxy_pass http://backend_server_address;
    }
}
``` 

## 7. **Blocking IP Addresses**

Prevent specific IPs or ranges from accessing the server:


```server {
    listen 80;
    deny 192.168.1.1;
    allow all;
}
```

## 8. **WebSocket Support**

Serve applications that need WebSocket:

nginxCopy code

```server {
    listen 80;
    location /ws/ {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
``` 

## 9. **Enabling Gzip Compression**

Reduce the size of the transferred data:

```http {
    gzip on;
    gzip_types text/plain application/xml text/css application/javascript;
}
``` 

## 10. **Securing with Basic Authentication**

Protect content with a password:
```
server {
    listen 80;
    location / {
        auth_basic "Restricted Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
``` 

These configurations cover many of the typical use-cases for NGINX, but the possibilities are vast. Each configuration block can be combined and adjusted as per your requirements, but be cautious about conflicts and redundancies. Always test the configuration using `nginx -t` before applying it to a live environment.

### Apply the configuration
```
sudo nginx -s reload
```
