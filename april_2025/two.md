# Kubernetes, Docker & DevOps Interview Questions & Answers

This document contains a list of common interview questions related to Kubernetes, Docker, Terraform, networking concepts, and general DevOps practices, along with concise answers.

---

## 1. What is a Pod in Kubernetes? Create a pod.yaml for a single-container pod running Nginx.

*   **Answer:** A Pod is the smallest and simplest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster and can contain one or more tightly coupled containers that share resources like networking (IP address) and storage volumes.

*   **`pod.yaml`:**
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest # Using latest for simplicity, specify version in production
        ports:
        - containerPort: 80
    ```

---

## 2. What is a Deployment in Kubernetes? Write a deployment.yaml for deploying 3 replicas of an Nginx container.

*   **Answer:** A Deployment is a Kubernetes controller that provides declarative updates for Pods and ReplicaSets. You describe a desired state in a Deployment object, and the Deployment Controller changes the actual state to the desired state at a controlled rate. Deployments are typically used for stateless applications and manage scaling, rolling updates, and rollbacks.

*   **`deployment.yaml`:**
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template: # Pod template starts here
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
    ```

---

## 3. What is a Service in Kubernetes, and what are the types of Services?

*   **Answer:** A Service is an abstraction that defines a logical set of Pods and a policy by which to access them (often referred to as a microservice). It provides a stable IP address and DNS name for a set of Pods, enabling loose coupling between application components. Even as Pods come and go, the Service IP/DNS remains constant.
*   **Types of Services:**
    *   **ClusterIP:** (Default) Exposes the service on a cluster-internal IP. Only reachable from within the cluster.
    *   **NodePort:** Exposes the service on each Node's IP at a static port. Allows external access via `<NodeIP>:<NodePort>`.
    *   **LoadBalancer:** Exposes the service externally using a cloud provider's load balancer. Creates a NodePort and ClusterIP service automatically, which the external load balancer routes to.
    *   **ExternalName:** Maps the service to the contents of an external DNS name (e.g., `foo.bar.example.com`), returning a CNAME record. No proxying involved.

---

## 4. When would you use each type of Kubernetes Service (ClusterIP, NodePort, LoadBalancer, ExternalName)?

*   **Answer:**
    *   **ClusterIP:** For internal communication between services within the cluster (e.g., backend service accessed by a frontend service).
    *   **NodePort:** For development/testing purposes or when you need direct external access to a specific node's port, but without a cloud load balancer (e.g., exposing a non-critical admin tool). Not generally recommended for production web traffic due to manual management and node dependency.
    *   **LoadBalancer:** The standard way to expose production applications externally when running on a cloud provider that supports it (AWS, GCP, Azure, etc.). Provides high availability and distributes traffic.
    *   **ExternalName:** To provide an internal service name alias for an external service, allowing applications within the cluster to use an internal name to refer to an external resource.

---

## 5. Write a simple Terraform script to provision a virtual machine on AWS.

*   **Answer:** This script provisions a basic EC2 instance. (*Note: Requires AWS provider configuration and potentially specifying an AMI ID available in your region*).
    ```terraform
    provider "aws" {
      region = "us-west-2" # Example region
    }

    resource "aws_instance" "example_vm" {
      ami           = "ami-0c55b159cbfafe1f0" # Example: Amazon Linux 2 AMI in us-west-2
      instance_type = "t2.micro"             # Example instance type

      tags = {
        Name = "HelloWorldVM"
      }
    }
    ```

---

## 6. Explain `port`, `targetPort`, and `nodePort` in a Kubernetes service.

*   **Answer:**
    *   **`port`**: The port number on which the Service itself is exposed *within the cluster*. Other pods in the cluster connect to the Service using this port and the Service's ClusterIP.
    *   **`targetPort`**: The port number on the *Pod* that the Service should forward traffic to. This is the port your container is actually listening on. It can be a number or a named port from the Pod definition.
    *   **`nodePort`**: (Only used in `NodePort` and `LoadBalancer` service types) The static port number (typically in the 30000-32767 range) exposed on the *Node's* IP address. External traffic hitting `<NodeIP>:<NodePort>` is routed to the Service's internal `port`, which then forwards to the Pod's `targetPort`.

---

## 7. How would you expose a Kubernetes application externally?

*   **Answer:** The main ways are:
    *   **`Service` type `LoadBalancer`:** Ideal for cloud environments. Provisions an external cloud load balancer that routes traffic to the service.
    *   **`Service` type `NodePort`:** Exposes the application on a specific port on all nodes. Suitable for testing or specific use cases, less ideal for production web traffic.
    *   **`Ingress`:** An API object that manages external access (primarily HTTP/HTTPS) to services within the cluster. It acts as a layer 7 reverse proxy, offering features like host-based routing, path-based routing, SSL termination, and load balancing. An Ingress Controller (like Nginx Ingress, Traefik) must be running in the cluster to fulfill the Ingress rules. This is the most common and flexible method for web applications.

---

## 8. What is Helm, and what are its components (Chart, Repository, Release)?

*   **Answer:** Helm is the package manager for Kubernetes. It helps you define, install, and upgrade even the most complex Kubernetes applications.
    *   **Chart:** A Helm package containing all the resource definitions (templates), metadata, and configuration values needed to run an application, tool, or service inside a Kubernetes cluster.
    *   **Repository:** A location where Charts can be stored and shared (like an Apt/Yum repo or Docker registry).
    *   **Release:** An instance of a Chart running in a Kubernetes cluster. A single Chart might be installed multiple times into the same cluster, each installation creating a new Release with its own name and configuration.

---

## 9. What is the difference between `EXPOSE` in a Dockerfile and `docker run -p`?

*   **Answer:**
    *   **`EXPOSE <port>` (in Dockerfile):** This is primarily documentation. It informs users (and potentially other tools/containers using linking) which ports the containerized application *intends* to listen on. It does **not** actually publish the port or make it accessible from the host.
    *   **`docker run -p <host_port>:<container_port>`:** This command *actively publishes* the container's port (`<container_port>`) to the host machine's network interface on a specific port (`<host_port>`). This makes the container's service accessible externally via the host's IP and the specified host port.

---

## 10. How do you run Nginx on a Linux server using Docker?

*   **Answer:** Use the `docker run` command:
    ```bash
    # Run Nginx in the background (-d), map host port 80 to container port 80 (-p 80:80),
    # and name the container "my-nginx" (--name my-nginx)
    docker run -d -p 80:80 --name my-nginx nginx:latest
    ```
    You can then access Nginx by browsing to the server's IP address or `http://localhost`.

---

## 11. Explain HTTP, HTTPS, TCP, and UDP with examples.

*   **Answer:**
    *   **TCP (Transmission Control Protocol):** A connection-oriented, reliable transport layer protocol. It guarantees delivery order and retransmits lost packets. Used when reliability is crucial.
        *   *Example:* Web browsing (HTTP/HTTPS), Email (SMTP), File Transfer (FTP).
    *   **UDP (User Datagram Protocol):** A connectionless, unreliable transport layer protocol. It's faster than TCP because it doesn't guarantee delivery or order. Used when speed is more important than perfect reliability.
        *   *Example:* DNS lookups, VoIP, online gaming, video streaming.
    *   **HTTP (Hypertext Transfer Protocol):** An application layer protocol for transmitting hypermedia documents (like HTML). It's the foundation of data communication for the World Wide Web. It typically runs over TCP port 80. It's stateless.
        *   *Example:* Accessing a website (`http://example.com`).
    *   **HTTPS (HTTP Secure):** HTTP layered over TLS/SSL (Transport Layer Security/Secure Sockets Layer). It provides encryption, authentication, and integrity for web communications. It typically runs over TCP port 443.
        *   *Example:* Securely logging into a bank website (`https://mybank.com`).

---

## 12. What is a Dockerfile? Write a basic Dockerfile for a Node.js application.

*   **Answer:** A Dockerfile is a text script that contains a series of instructions on how to build a Docker image. It automates the process of creating an image by specifying the base image, adding dependencies, copying files, setting environment variables, exposing ports, and defining the command to run when a container starts.

*   **Basic Node.js Dockerfile:** (Assumes `package.json` exists)
    ```dockerfile
    # Use an official Node.js runtime as a parent image
    FROM node:18-alpine # Using a specific LTS version and alpine for smaller size

    # Set the working directory in the container
    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json (if available)
    COPY package*.json ./

    # Install app dependencies
    RUN npm install
    # If you are building your code for production
    # RUN npm ci --only=production

    # Bundle app source inside Docker image
    COPY . .

    # Make port 8080 available to the world outside this container
    EXPOSE 8080

    # Define the command to run your app using CMD which defines your runtime
    CMD [ "node", "server.js" ] # Assuming your entry point is server.js
    ```

---

## 13. What is a base image in Docker? Which base image would you use for Python or Node.js?

*   **Answer:** A base image is the starting point for a Dockerfile. It's usually an image containing an operating system (like Alpine, Ubuntu, Debian) or a specific runtime environment (like Node.js, Python). Subsequent instructions in the Dockerfile add layers on top of this base image.
*   **Choosing Base Images:**
    *   **Python:** `python:<version>-slim` (e.g., `python:3.10-slim`) is often a good balance, providing necessary libraries without the full OS bloat. `python:<version>-alpine` is even smaller but may require compiling dependencies or installing extra packages due to its minimal nature (musl libc vs glibc).
    *   **Node.js:** `node:<version>-alpine` (e.g., `node:18-alpine`) is very popular for its small size. `node:<version>-slim` is an alternative if Alpine causes compatibility issues with native modules. Always prefer specific versions over `latest`.

---

## 14. How do you check for open ports on a Linux system?

*   **Answer:** Several commands can be used:
    *   **`ss` (Socket Statistics - preferred modern tool):**
        ```bash
        # Show listening TCP and UDP sockets with process information numerically
        ss -tulnp
        ```
    *   **`netstat` (Network Statistics - older tool, may not be installed by default on newer systems):**
        ```bash
        # Show listening TCP and UDP sockets with process information numerically
        netstat -tulnp
        ```
    *   **`lsof` (List Open Files):** Can check specific ports.
        ```bash
        # Show processes listening on a specific port (e.g., 80)
        sudo lsof -i :80
        ```

---

## 15. What are the benefits of using a firewall?

*   **Answer:** Firewalls act as a barrier between a trusted internal network and untrusted external networks (like the internet). Key benefits include:
    *   **Access Control:** Blocks unauthorized access attempts from outside.
    *   **Traffic Filtering:** Allows defining rules to permit or deny specific types of traffic (based on ports, protocols, source/destination IPs).
    *   **Threat Prevention:** Can help mitigate certain types of attacks like port scanning and denial-of-service (DoS).
    *   **Network Segmentation:** Can enforce security policies between different segments of an internal network.
    *   **Logging and Monitoring:** Provides logs of allowed and blocked traffic, aiding in security audits and incident response.

---

## 16. What is the use of Ingress and Ingress Controller in Kubernetes?

*   **Answer:**
    *   **Ingress:** An API object in Kubernetes that defines rules for routing external HTTP and HTTPS traffic to internal Services. It allows you to configure rules based on hostname (virtual hosting) or URL path. It acts as a specification for Layer 7 routing.
    *   **Ingress Controller:** A pod/deployment running in the cluster that *fulfills* the rules defined in Ingress resources. It's typically a reverse proxy and load balancer (like Nginx, Traefik, HAProxy). It watches the Kubernetes API for Ingress objects and configures itself accordingly to route external traffic to the correct services based on the defined rules.
    *   **Use:** They work together to provide sophisticated external access management, including SSL/TLS termination, name-based virtual hosting, path-based routing, and load balancing for web applications running in Kubernetes, often consolidating multiple services under a single external IP address.

---

## 17. Explain the Kubernetes controllers: Deployment, StatefulSet, ReplicaSet, and DaemonSet.

*   **Answer:** These are core Kubernetes controllers that manage Pods:
    *   **ReplicaSet:** Ensures that a specified number of identical Pod replicas are running at any given time. It's the simplest controller for ensuring availability and scaling. Primarily used *by* Deployments, not directly by users often.
    *   **Deployment:** Manages ReplicaSets to provide declarative updates to applications. It handles rolling updates (zero-downtime updates), rollbacks, scaling, and pausing of stateless applications. This is the most common controller for managing application Pods.
    *   **StatefulSet:** Manages the deployment and scaling of stateful applications. It provides guarantees about the ordering and uniqueness of Pods, stable network identifiers (DNS names), and stable persistent storage per Pod. Used for databases, message queues, etc.
    *   **DaemonSet:** Ensures that all (or some specified) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed, those Pods are garbage collected. Useful for cluster-level agents like logging collectors (Fluentd, Logstash), monitoring agents (Prometheus Node Exporter), or network plugins.

---

## 18. What is the difference between Deployment and ReplicaSet?

*   **Answer:** A ReplicaSet ensures a specific number of identical pods are running. A Deployment is a higher-level controller that *manages* ReplicaSets to provide sophisticated features like rolling updates and rollbacks. When you update a Deployment (e.g., change the container image), it creates a *new* ReplicaSet with the updated configuration and gradually scales down the *old* ReplicaSet while scaling up the new one. Users typically interact with Deployments, not directly with ReplicaSets for application management.

---

## 19. What are Kubernetes Probes (Liveness, Readiness, Startup)?

*   **Answer:** Probes are health checks performed periodically by the Kubelet (the agent running on each node) on containers within a Pod.
    *   **Liveness Probe:** Checks if a container is still running/alive. If the liveness probe fails (e.g., the application deadlocked), the Kubelet kills the container, and the container is restarted subject to its restart policy. *Purpose: Restart unhealthy containers.*
    *   **Readiness Probe:** Checks if a container is ready to serve traffic. If the readiness probe fails, the container's Pod IP address is removed from the endpoints of any matching Services. Traffic is not sent to it until the probe succeeds again. *Purpose: Prevent sending traffic to Pods that are running but not yet ready (e.g., still initializing, overloaded).*
    *   **Startup Probe:** Checks if a container application has started successfully. If configured, all other probes are disabled until the startup probe succeeds. If the startup probe fails beyond its configured threshold, the container is killed and restarted (similar to a failed liveness probe). *Purpose: Allow applications with slow start times enough time to initialize before liveness probes start killing them prematurely.*

---

## 20. What is the difference between Stateful and Stateless applications? Give examples.

*   **Answer:**
    *   **Stateless Application:** Does not save client data generated in one session for use in the next session with that client. Each request is handled independently, without relying on any stored state from previous requests on that specific server instance. They are easy to scale horizontally because any instance can handle any request.
        *   *Examples:* Web servers serving static content (Nginx, Apache), API gateways, most web application frontends, function-as-a-service (FaaS).
    *   **Stateful Application:** Remembers data about past interactions or the state of the system. Requires stable storage and often stable network identifiers, as subsequent requests might need to go to the same instance or access specific persistent data.
        *   *Examples:* Databases (MySQL, PostgreSQL, MongoDB), message brokers with persistent queues (Kafka, RabbitMQ), applications that store user session data locally.

---

## 21. What are Namespaces in Kubernetes?

*   **Answer:** Namespaces provide a mechanism for isolating groups of resources within a single Kubernetes cluster. They act like virtual clusters within the physical cluster.
    *   **Uses:**
        *   **Scope for Names:** Resource names (like Pods, Services, Deployments) only need to be unique *within* a namespace, not across the entire cluster.
        *   **Resource Quotas:** Administrators can assign resource quotas (CPU, memory limits) on a per-namespace basis.
        *   **Access Control:** Role-Based Access Control (RBAC) policies can be scoped to specific namespaces, allowing different teams or users to have different permissions in different namespaces.
        *   **Environment Separation:** Commonly used to separate environments like development, staging, and production within the same cluster.

---

## 22. What is Port Forwarding in Kubernetes?

*   **Answer:** Port forwarding in Kubernetes (`kubectl port-forward`) allows you to establish a direct, secure tunnel from your local machine to a specific port on a Pod or Service running inside the cluster. It maps a local port on your machine to a port within the cluster resource.
    *   **Use:** Primarily used for debugging purposes, allowing developers to directly access an application running in a Pod as if it were running locally, without needing to expose it via a Service externally. For example, accessing a database pod directly from your local database client using `kubectl port-forward <pod-name> <local-port>:<pod-port>`.

---
