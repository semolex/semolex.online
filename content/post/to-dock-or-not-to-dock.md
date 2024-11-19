+++
date = '2024-11-19T15:06:00+02:00'
draft = true
title = 'To Dock or Not to Dock'
tags = ['docker', 'deployment', 'devops', 'digitalocean', 'heroku', 'aws']
+++

## Introduction

Hey there! If you're a developer, you've probably worked with resource-constrained environments – you know, those basic [DigitalOcean](https://www.digitalocean.com/pricing/droplets) droplets, [Heroku](https://www.heroku.com/dynos) free dynos, or [AWS](https://aws.amazon.com/free) free tier. While they might seem limiting, they can be really capable if tuned.
<!--more-->

### The Reality of Resource-Constrained Environments

Let's get real about these environments for a moment:

- They can actually run many applications effectively (especially static websites)
- They're either free or very budget-friendly
- They make excellent sandboxes for learning and development
- They teach us to be mindful of resource usage (CPU, memory, disk space)

### The Docker Dilemma

So, here's the big question: Should we run Docker in these environments? It's not a straightforward yes or no – let's break it down.

#### The Good Parts:
1. **Portability**:
    - Build once, run anywhere (well, almost!)
    - Easy deployment with well-prepared `Dockerfile` and `docker-compose`.yml

2. **Isolation**:
   - Your app lives in its own little world
   - Better security through containerization
   - No more "but it works on my machine!" situations (almost :smile:)

3. **Dependency Management**
   - Run multiple `Node.js` versions without conflicts
   - Mix `Python 2` and `Python 3` applications easily
   - Keep your `PostgreSQL` 12 and 14 instances separate

#### The Not-So-Good Parts

1. **Resource Overhead**
   - Docker daemon needs its own resources
   - Each container adds some memory overhead
   - Images take up valuable disk space

2. **Debugging Challenges**

   - Container logs can be less straightforward
   - Network issues can be trickier to diagnose
   - Performance bottlenecks might be harder to identify

3. **Setup Complexity**

   - Initial Docker configuration takes time
   - Volume permissions can be tricky
   - "It works in Docker" doesn't always mean it works everywhere

**Example Dockerfile**    
```dockerfile
   # Simple example of a lightweight Node.js app
    FROM node:alpine
    WORKDIR /app
    COPY package*.json ./
    RUN npm ci --only=production
    COPY . .
    CMD ["npm", "start"]
```

### Real-World Approaches

#### Hybrid approach

Here's a practical example from my own experience. 
This very website (as of pub date) runs as containerized app, but to be honest I should better switch it to use mix of containerized and non-containerized services:

```yaml
# Partial docker-compose.yml example
services:
  mysql:
    image: mysql:8
    volumes:
      - mysql_data:/var/lib/mysql
  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data
```

In this approach the main `Wagtail CMS` application runs directly on the host, while `MySQL` and `Redis` run in containers. This gives us:

- Easier main application debugging
- Quick configuration tweaks without rebuilds
- Isolated database and cache services

Running web server (reverse proxy, static files etc.) directly might be a good idea too. It works fine in `Docker` but in case of loosing persistent volume and giving up your _SSL certificates_ it is just easier. 
You can also find it a little bit easier to point your multiple apps from the same place without "attaching" server to one specific docker-compose.yaml.

I **_do understand_** that many engineers will not agree, but it is obviously easier.


#### No Docker

If you think that you need containerization only for the sake of containerization, you might be better off without it.
If there is more overhead than benefit - do you really need it?
You can always keep `Dockerfile` for development (no junk on system, right?) and testing, but deploy your app directly to the server.

### Smart Deployment Strategies

Consider these approaches for resource-constrained environments:

**Minimal Base Images:**

* Use alpine variants when possible
* Multi-stage builds to reduce final image size
* Only install production dependencies
* Remove unnecessary files and directories

**Selective Containerization:**

* Containerize services that might conflict (databases, caches)
* Keep frequently updated applications on the host
* Run web servers directly on the host if serving multiple apps


### The Bottom Line

Docker isn't a magic solution – it's just another tool in our developer toolkit. Don't feel pressured to containerize everything just because it's trendy. Consider your specific needs:

✅ Use Docker when:

* You need to run multiple apps with conflicting dependencies
* You want a clean, reproducible development environment
* You're planning to scale or move your application later

❌ Skip Docker when:

* You're running a single, simple application
* Resources are extremely limited
* The overhead isn't worth the benefits

Remember: the best solution is the one that works for your specific situation. Sometimes that means embracing Docker fully, sometimes it means running everything directly on the host, and often it means finding a sweet spot somewhere in between.

### A Final Tip

Start small and measure the impact. Try it by yourself, really! There is no "_official approach_".