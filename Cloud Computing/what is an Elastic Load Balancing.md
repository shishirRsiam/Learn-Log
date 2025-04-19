**Elastic Load Balancing (ELB)** is a service provided by **Amazon Web Services (AWS)** that automatically distributes incoming application traffic across multiple targets—such as **EC2 instances, containers, IP addresses**, and **Lambda functions**—in one or more Availability Zones.

### 🔍 Why it's important:
- It **improves availability** by distributing traffic evenly.
- Helps your application **scale** automatically with traffic.
- Adds **fault tolerance** by redirecting traffic away from unhealthy targets.

---

### 🧠 Types of Elastic Load Balancers:

1. **Application Load Balancer (ALB)** – Layer 7
   - Best for **HTTP/HTTPS** traffic.
   - Supports path-based and host-based routing.
   - Good for **microservices** and **containers**.

2. **Network Load Balancer (NLB)** – Layer 4
   - Handles **TCP, UDP, TLS** traffic.
   - Designed for **high-performance, low-latency** scenarios.
   - Suitable for millions of requests per second.

3. **Gateway Load Balancer (GWLB)**
   - Used to **deploy, scale, and manage third-party virtual appliances**.
   - Works well with **firewalls, intrusion detection systems**, etc.

4. **Classic Load Balancer (CLB)** – Legacy
   - Supports both Layer 4 and Layer 7.
   - Now mostly replaced by ALB and NLB.

---

### ⚙️ Key Features:
- **Health checks**: Routes traffic only to healthy targets.
- **Auto scaling**: Works with Auto Scaling groups.
- **SSL termination**: Offloads SSL decryption from your app.
- **Sticky sessions**: Keeps a user session on the same instance.

---

### Example Use Case:
You have a web app hosted on multiple EC2 instances. An **Application Load Balancer** can route traffic to different instances based on the URL path (e.g., `/api` vs `/admin`), making the app more efficient and resilient.
