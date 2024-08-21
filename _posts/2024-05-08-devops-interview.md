---
title: "I went to an interview for Dev(Sec)Ops position. I should have answered better for these questions."
date: 2024-08-05 00:00:00 +0700
categories: [CareerPath, DevOps]
tags: [interview, devops, networking, system-design, k8s, security]
image:
  path: ../picture/devops/dso.png
  alt: DevSecOps pipeline.
---


## 1. Introduction

Interviews can be challenging, especially when aiming for a specialized role like Dev(Sec)Ops. Recently, I attended many interviews for a Dev(Sec)Ops position, and while it was a valuable learning experience, I realized there were some questions I could have answered more effectively. In this post, I'll share the questions I faced, and how I could have improved them. Whether you're preparing for a similar role or just curious about the Dev(Sec)Ops interview process, I hope my experience provides useful insights. 

## 2. Networking

### 2.1 When you open a browser, type in an URL and search it, what will happen below the system, especially the workflow in OSI model.

- I prefer breaking down this process into some smaller processes. First is the request flow.


1. Application layer (Layer 7)

- The browser takes the URL (e.g., www.example.com) and prepares an HTTP or HTTPS request.
Before sending the request, the browser needs the IP address of the domain, which is the DNS Query process. 

- The browser should checks its cache to see if it already has the IP address of the domain.
If not found, it sends a DNS query to the configured DNS resolver (often the ISP's DNS server or a public DNS server like Google’s 8.8.8.8, which based on personal setup).

- The DNS resolver checks its cache. If the IP address is not cached, it performs a recursive query to find the IP address. This involves querying root DNS servers, top-level domain (TLD) servers, and authoritative DNS servers for the domain.

![Recurvise and Iterative DNS Query.](../picture/devops/dns.png)

- Once the DNS resolver obtains the IP address, it sends it back to the browser. The browser can now proceed with the HTTP or HTTPS request using the resolved IP address.

- The headers are created and interpreted at this layer. The browser or web server adds these headers to the HTTP messages to convey metadata about the request or response.

![HTTP header.](../picture/devops/http-header.png)

1. Presentation Layer (Layer 6):
The data is encrypted if using HTTPS. The browser translates the URL into a format that can be understood by the lower layers.
1. Session Layer (Layer 5):
A session is established between your computer and the web server. This layer manages the dialog control and keeps the session alive.
1. Transport Layer (Layer 4):
The TCP breaks down the data into **segments**. It ensures reliable transmission, error-checking, and flow control. For HTTPS, TLS operates here to encrypt and secure the data.
1. Network Layer (Layer 3):
The IP addresses and routes the data packets to the destination server. It determines the best path for data transfer across networks.
1. Data Link Layer (Layer 2):
The data packets are **framed** and addressed using **MAC** addresses. This layer handles error detection and correction from physical layer issues.
1. Physical Layer (Layer 1):
The actual transmission of raw data bits occurs over the physical medium. The signals are sent to the network interface and onto the internet.

Response flow.

Reception:
The client receives the frames at the Physical Layer and passes them up through the Data Link Layer, Network Layer, and Transport Layer.

Reassembly:
At the Transport Layer, TCP segments are reassembled into the complete HTTP response.

Decryption:
If HTTPS is used, the Presentation Layer decrypts the response.

Parsing:
The Application Layer parses the HTTP response headers and body.

Rendering:
The browser processes the HTML content and renders the webpage for the user to view.

## 3. Architecture/System Design

### 3.1 The interviewer want to design a system using for hosting a website, describe some components in the desired system (what should you have in a basic hosting system?).

- I prefer using a microservices system. On the interview day I focused too much about the Security function and Architecture but forget about some basic components like WAF, IDPS, Database...

- This is my suggestion architecture. Also in my interview, I also suggested about Function-based Access Control, you can read more [here](https://arxiv.org/pdf/1609.04514) and OAuth2.0 framework, which can be found at [here](https://datatracker.ietf.org/doc/html/rfc6749).

![Basic microservices architecture.](../picture/devops/architecture.png)

## 4. Kubernetes
### 4.1 How network works in K8s (how can each pod talk to eachother)

- I may have write alot of YAML config file for my cluster but I never thought about this problem.
- Network in K8s is kinda simple. First, checkout a K8s network diagram I found online

![Kubernetes network diagram.](../picture/devops/k8s-net.png)

- The K8s network allocates IP addresses, assign DNS names and maps ports automatically into your Pod. Usually the DNS based on the namespace set up on your config file. Example as I want to call an API endpoint of my BE service, my endpoint would look something like this below: 

```yaml
AUTHO_URL: "http://autho.default.svc.cluster.local:5013/api-autho/"
AUTHEN_URL: "http://authen.default.svc.cluster.local:5012/api-authen/"
BLOG_URL: "http://blog.default.svc.cluster.local:7777/api-blog/"
PROD_URL: "http://products.default.svc.cluster.local:8888/api-product/"
```
- The prototype for Pod and Service DNS are:

```
Pod – pod-ip-address.pod-namespace-name.pod.cluster-domain.example 

Service – service-name.service-namespace-name.svc.cluster-domain.example
```
- At a high level, the Kubernetes network model assigns a unique IP address to each Pod within your cluster. This allows Pods to communicate directly using their IP addresses, eliminating the need for NAT or additional configurations.

**Conclusion**: Each Pod receives its own IP address and network namespace. Nodes use a root network namespace to bridge Pod interfaces, enabling Pod-to-Pod communication across Nodes without NAT, which simplifies networking and enhances portability. The cluster-level network layer manages routing, so manual Pod port binding to Nodes is unnecessary, though possible with hostPort if required.

**Additional Information:**

1. How K8s allocates IP addresses: assigns IP addresses to Pods using the Classless Inter-Domain Routing (CIDR) system, defining the subnet of available IP addresses. Each Pod gets an address from the cluster's CIDR range, which must be specified when setting up the cluster's networking layer. Many Kubernetes networking plugins support IP Address Management (IPAM), allowing manual assignment of IP addresses, prefixes, and pools for advanced network management.
1. Kubernetes network isolation with Network Policies: By default, Kubernetes allows all Pods to communicate with each other, posing a security risk for clusters shared among different apps, environments, teams, or customers.
**Kubernetes Network Policies** are API objects that let you specify allowed ingress and egress routes for your Pods. For example, a policy can block traffic to Pods labeled **app-component: database**, unless it originates from Pods labeled **app-component: api**.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app-component: database
  policyTypes:
    - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            app-component: api
```

## 5. Linux
### 5.1 What is the differences between Linux Kernel and Linux Distro?

**Linux Kernel**
- The Linux kernel is the core component of the Linux operating system.
- It manages system resources and hardware, enabling communication between hardware and software.
- Handles memory management, process management, and hardware interaction.

**Shell**
- A command-line interface that allows users to interact with the operating system.
- It interprets and executes user commands, scripts, and programs.
- Common shells include Bash (Bourne Again Shell), Zsh, and Fish.
- rovides an interface for users to run commands, manage files, and execute scripts.

**Linux Distribution (Distro)**
- A Linux distribution is a packaged version of the Linux operating system, combining the kernel, shell, and various software applications.
- Includes the Linux kernel, system libraries, utilities, and often a package management system.
- Ubuntu, Fedora, CentOS, and Debian.
- Provides a complete operating system tailored for different user needs, with pre-configured settings and bundled applications.

**Summary**
- Kernel: The core of the OS, managing hardware and system resources.
- Shell: The user interface for command execution and interaction with the OS.
- Distro: A complete OS package that includes the kernel, shell, and additional software.





## References
https://spacelift.io/blog/kubernetes-networking


