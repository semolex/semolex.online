+++
date = '2024-11-20T15:36:58+02:00'
draft = false
title = 'Give Caddy a Try'
tags = ['caddy', 'webserver', 'https', 'tls', 'reverse-proxy', 'docker', 'deployment', 'devops']
+++

## Introduction

Not every developer in the modern era knows or needs to configure a webserver itself.
And even if you do, you might not want to spend hours setting up a huge, complex webserver like Apache or Nginx.
That's where [Caddy](https://caddyserver.com) comes in.
<!--more-->

### What is Caddy?
_Officially_, Caddy is a "powerful, extensible platform to serve your sites, services, and apps, written in Go".

_Personally_, I like to think of it as a cool, modern webserver that's easy to set up and use.
It is especially great for small projects, personal websites, and APIs (it does not mean it can't handle bigger projects).
I fell in love with it when I just needed something to serve my static files and reverse-proxy to my backend services.
Indeed, I could have used Nginx or Apache, but I really wanted to try something else, probably something "lighter".
I was also looking at [lighttpd](https://www.lighttpd.net), but I tried Caddy. And I liked it.
By the way, the most known feature of Caddy is its automatic HTTPS setup using [Let's Encrypt](https://letsencrypt.org) certificates.


### Caddyfile

The configuration of Caddy is done via a `Caddyfile`.
Official documentation is available [here](https://caddyserver.com/docs/caddyfile).
It also supports `JSON` configuration and looks like it is a preferred way according to the documentation.
You can also utilise [REST API](https://caddyserver.com/docs/api) to configure Caddy.
However, you will see most examples in the `Caddyfile` format (which internally is converted to JSON).
My opinion is following:
- JSON is verbose but most people are familiar with it
- Caddyfile is more _classic_ way to configure a webserver
- REST API is great for automation and integration with other tools

While I think that some common config language like `YAML` or `TOML` would be a better choice (maybe I am wrong and having +1 specific format is better), I am fine with `Caddyfile`.
JSON seems to be more powerful and flexible, but I am not sure if I need that flexibility for my personal projects.
Too much words about configuration, let's see an example:

```Caddyfile
# The most basic Caddyfile
localhost

respond "Hello, world!"
```

Probably this is the simplest `Caddyfile` you can have.
It listens on `localhost` and responds with `Hello, world!` to every request.
This is a great way to test if Caddy is working.
But you can do much more with Caddy.
Assume you want to serve a static website.
You can do it with a single line in `Caddyfile`:

```Caddyfile
# Serve static files
localhost   # or your domain
root * /path/to/your/static/files
```

This will serve your static files from `/path/to/your/static/files` directory.

And if you have few domains and want to serve different content for each of them, you can do it like this:

```Caddyfile
# Serve different content for different domains
domain1.com {
    root * /path/to/domain1/files
}

domain2.com {
    root * /path/to/domain2/files
}
```

Note that you need to have `{}` blocks if you want to configure multiple sites.

Official:
`Only the first line can be the address(es) of the site, and then all the rest of the file has to be directives for that site.`

### Reverse Proxy

One of the most common use cases for me is to use Caddy as a reverse proxy.
I found it very easy to set up.
Here is an example of how you can set up a reverse proxy to your backend service:

```Caddyfile
# Reverse proxy to backend service
localhost {
    reverse_proxy localhost:8080
}
```

This will forward all requests to `localhost:8080` to your backend service.
And indeed, you can additionally serve static files or do anything else in the same `Caddyfile`:
```Caddyfile
localhost {
    
    # Serve static files
    handle_path /static/* {
        root * /app/staticfiles/
        file_server
    }
    # Serve media files
    handle_path /media/* {
        root * /app/media/
        file_server
    }

    # Proxy all other requests to Gunicorn
    reverse_proxy /* app:8000
    
    # Enable gzip compression
    encode gzip 
}
```

### Automatic HTTPS
Caddy is known for its automatic HTTPS setup.
And it does it by default.
You don't need to do anything to get your site served over `HTTPS`.
Just make sure your domain is pointing to your server and Caddy will take care of the rest.
You can also configure it to use your own certificates or disable `HTTPS` if you want.
Note that even on `localhost` Caddy will serve your site over `HTTPS`.
So if this is not what you want, you need to explicitly point Caddy to use `HTTP`:
```Caddyfile
# Explicitly use HTTP
http://localhost {

    reverse_proxy /* app:8000

    encode gzip  # Enable gzip compression
}
```
Yon can use your own certificates like this:
```Caddyfile
# Use your own certificates
yourdomain.com {
    tls /path/to/your/cert.pem /path/to/your/key.pem
    reverse_proxy localhost:8080
}
```

### Other Features
Caddy has a lot of other features. There is too much to describe in a single post.
For example, you can use `templates` in your `Caddyfile` [to generate dynamic content](https://caddyserver.com/docs/caddyfile/directives/templates#templates).

You can also use `plugins` [to extend Caddy's functionality](https://caddyserver.com/docs/modules/).
Logs, matchers, rate limiting, and many other features are available.

You can use Caddy in so many ways, and it can be as complex as you need it to be.
What is good is that ou can start _simple_. And defaults are just good.

### Conclusion
Give Caddy a try.