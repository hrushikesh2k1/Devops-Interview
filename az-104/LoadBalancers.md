# Azure Load Balancing

## What is a Load Balancer?
A load balancer distributes incoming network traffic across multiple servers so that:
- No single server is overloaded
- Performance increases
- Downtime is reduced
- High availability is maintained

Azure provides four different ways to balance workloads.

---

## The Four Load Balancing Services in Azure

| Service | Purpose |
|--------|--------|
| Azure Load Balancer | General traffic distribution |
| Application Gateway | Web application load balancing |
| Azure Front Door | Global CDN and caching |
| Azure Traffic Manager | DNS-based global traffic routing |

---

## 1. Azure Load Balancer (Standard)

This is the most flexible and general-purpose load balancer.

### What it supports
- Web servers  
- Application servers  
- FTP servers  
- Database servers  
- Any TCP/UDP service  

It works at Layer 4 (Transport Layer), meaning it routes traffic using IP and Port only.

### Use this when
You need basic load balancing inside Azure for any kind of service.

---

## 2. Application Gateway

Application Gateway is designed only for web applications.

### Works at
Layer 7 (Application Layer), meaning it understands:
- URLs
- Hostnames
- Paths like `/login`, `/cart`

### Key Features
- Web Application Firewall (WAF) for security
- SSL/TLS termination for HTTPS
- Path-based routing

### Network Requirement
It requires a Virtual Network with at least two subnets.

### Use this when
You are hosting:
- Web apps
- APIs
- HTTPS websites

---

## 3. Azure Front Door

Azure Front Door is used to improve website speed for global users.

### Problem it solves
If your web server is in India and users access it from the USA or Europe, the website becomes slow.

### What Front Door does
It creates cached copies of your website in Azure data centers around the world.

So users get content from the nearest data center, which:
- Reduces latency
- Increases speed
- Improves user experience

No need to deploy servers in every country.

---

## 4. Azure Traffic Manager

Traffic Manager uses DNS-based routing.

### What it does
It directs users to the nearest or healthiest data center.

### Example
If you have:
- A website in India
- A website in the USA

Traffic Manager sends:
- Indian users to the India server
- US users to the USA server

This improves performance and availability.

---

## Interview Question

**How many ways are there to do load balancing in Azure?**

Answer:
Azure provides four load balancing solutions:
- Azure Load Balancer
- Application Gateway
- Azure Front Door
- Azure Traffic Manager

---

## Quick Comparison

| Feature | Load Balancer | Application Gateway | Front Door | Traffic Manager |
|--------|---------------|--------------------|------------|----------------|
| OSI Layer | Layer 4 | Layer 7 | Global CDN | DNS |
| URL-based routing | No | Yes | Yes | No |
| Web Application Firewall | No | Yes | No | No |
| Global acceleration | No | No | Yes | Yes |
| Works for non-web apps | Yes | No | Yes | Yes |

---

## Exam Tip

- Web + HTTPS + Security → Application Gateway  
- Global website speed → Azure Front Door  
- Multiple active regions → Traffic Manager  
- General load balancing → Azure Load Balancer  

   
