+++
date = '2024-11-22T15:25:36+02:00'
draft = false
title = 'Hiding An App Behind The Reverse Proxy'
tags = ['caddy', 'webserver', 'https', 'tls', 'reverse-proxy', 'docker', 'deployment', 'devops']
+++

## Introduction
If you just started with web development, you might have heard about a reverse proxy.
But what is it? And why would you need it?
Let's dive into it.
<!--more-->

### Why Reverse Proxy?
A reverse proxy is a server that sits between clients and backend servers.
A little bit of context:

If you tried to run your app locally, you probably used a built-in development web server bundled with frameworks like `Flask`, `Django`, or `Express`.
It's great for development, but it's not suitable for production.

Why? Because it's not secure, it's not scalable, and it's not performant.

Then you find out about "_app servers_" like `Gunicorn`, `uWSGI`, `Puma` or `PM2`.
Technically it is an upgraded version of the built-in server, full of features and optimizations.
They are capable of handling more requests, they can serve static files, they can run multiple workers, they can be managed by process managers, etc.

However, you might need something more - a reverse proxy.

Reverse proxy adds another layer of security, scalability, and performance:

* It can handle SSL termination (Caddy can automatically get certificates for you!), load balancing, caching, and prevent additional load on your app before traffic reaches it.
* It can also serve static files, compress responses, and cache responses.
* And you can have a single entry point for multiple apps running on different ports or even different servers.
* And more!

### When To Use It?
Let's answer this question with a few examples:

1. You are using a single server to host multiple apps.
2. You want to serve static files from a different location.
3. You want to use different domains for different apps or the same app with different domains :smile:.
4. You need full control over the traffic, routing, security.
5. You need a single entry point for multiple services.

Also, there is a concept of [API Gateway](https://www.redhat.com/en/topics/api/what-does-an-api-gateway-do) which is a more advanced version of a reverse proxy.
Therefore, some services like `Kong`, `Traefik`, `Ambassador`, `Envoy` are more than just a reverse proxy.

### How To Use It?
There are many reverse proxies available, but I will focus on [Caddy](https://caddyserver.com) in this post.
Why Caddy? Because it's easy to set up, it's modern, it's powerful, and it's free. And I love it!

Off-topic: I also would like to try `Traefik` - it seems to be a great tool for microservices.

Here is a simple example of how to use Caddy as a reverse proxy:

```Caddyfile
# Caddyfile
# When I say "youradress" I mean your IP address or domain name
# You can also use localhost if you are running Caddy on the same machine as your app
# BTW I switched "admin" off - Caddy includes an admin panel by default
{
    admin off
}

example.com {
    # Configure logging
    log {
        output file /data/logs/access.log {
            roll_size 100MB
            roll_keep 7
            roll_keep_for 20d
        }
        format json
        level INFO
    }

    log {
        output file /data/logs/error.log {
            roll_size 100MB
            roll_keep 7
            roll_keep_for 20d
        }
        format json
        level ERROR
    }
    # Serve static files from /static and /media
    # You might find it useful to serve them directly from Caddy
    handle_path /static/* {
        root * /app/staticfiles
        file_server
        header Cache-Control "public, max-age=31536000"
        header !ETag
    }

    handle_path /media/* {
        root * /app/media
        file_server
        
        # Block sensitive files
        # Even if you have them in .gitignore, it's better to block them.
        @blocked {
            path *.php *.py *.env *.config *.sh *.bash  
        }
        respond @blocked 403
    }

    # Proxy all other requests to your app
    reverse_proxy /* youradress:8000 {
        # Only valid/necessary headers
        header_up X-Real-IP {remote_host}
        header_up X-Forwarded-For {remote_host}
        header_up X-Forwarded-Proto {scheme}

        # If you need timeouts, they should be configured in transport
        transport http {
            read_timeout 30s
            write_timeout 30s
            dial_timeout 30s
        }
    }

    # Optional security headers
    # I found it useful to have them in one place
    # Especially if you have multiple sites
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
        X-XSS-Protection "1; mode=block"
        Referrer-Policy "strict-origin-when-cross-origin"
        Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=()"
        Content-Security-Policy "
        default-src 'self';
        script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.youtube.com https://*.googleapis.com;
        style-src 'self' 'unsafe-inline';
        img-src 'self' data: https: blob:;
        font-src 'self' data:;
        connect-src 'self';
        frame-src 'self' https://www.youtube.com https://youtube.com https://youtu.be;
        media-src 'self';
        base-uri 'self';
        form-action 'self'
        "
        # Remove server information
        -Server
        -X-Powered-By
    }
    # Enable gzip compression
    encode {
        gzip
        minimum_length 1000 # Only compress responses >1KB
    }
}
```

### Conclusion
You might find it useful to have a reverse proxy in front of your app.
I also encourage you to try many of them to find the one that suits you best.
To be honest I was amazed that there are so many options available.
I came from the world where  `Nginx` and `Apache` were the only options.