# Capacity Scaling in AWS

Businesses often experience peak seasons when user activity surges, requiring applications to handle increased loads. A prime example is e-commerce websites, which see regular traffic but encounter 10 to 20 times more visitors during peak shopping periods like Black Friday. This poses a capacity planning challenge, as the workload will drastically rise for a few months each year.

In traditional on-premises data centers, acquiring additional hardware takes 4 to 6 months to become application-ready. This necessitates careful capacity planning to avoid either idle resources or compromised user experiences. Public cloud services offer an ideal solution, enabling easy adjustment of resources—such as computing power and storage capacity—on-demand. This ensures the ability to accommodate sudden spikes without long delays or unnecessary idle resources.

## Architecture Scaling

![Scaling three-tier architecture](/architecture-diagram/Scaling-three-tier-architecture.png)

In a modern three-tier architecture for an e-commerce website, like the one depicted in the architecture diagram, several components play key roles in achieving elasticity and scalability:

- **Virtual Servers (Amazon EC2):** These are used for hosting web and application servers, which handle user requests. The load balancer directs incoming traffic to these servers.

- **Database (Amazon RDS):** This component manages the storage and retrieval of data. It plays a critical role in the application's functionality and needs to be scalable as well.

- **Load Balancer (Amazon ELB):** Responsible for distributing incoming traffic across multiple instances of web and application servers. This helps balance the load and ensure optimal performance.

- **DNS Server (Amazon Route53):** Manages domain names and directs users to the appropriate resources, contributing to efficient traffic routing.

- **CDN Service (Amazon CloudFront):** This serves static content like images and videos to users from locations close to them, improving load times and reducing the load on the main servers.

- **Network Boundary (VPC):** Provides isolation and security for the resources within the architecture, ensuring a safe environment for application operation.

- **Object Store (Amazon S3):** Used for storing static assets like images and videos, which can be accessed quickly and efficiently.

In this architecture, as depicted, user requests are first directed to the load balancer, which then forwards the traffic to the web servers. To handle varying levels of user activity, auto-scaling is employed. When user traffic surges, more web and application servers are automatically added to the fleet to handle the increased load. Similarly, during periods of low demand, extra servers are removed. Auto-scaling decisions can be based on factors such as CPU and memory utilization. For example, new servers can be added if CPU utilization surpasses 60%, and servers can be removed if it drops below 30%.

Furthermore, the architecture recognizes the importance of scaling storage as well. With the ever-increasing size of data, particularly static content like images and videos, storage scaling becomes vital. This component warrants particular attention to ensure seamless storage expansion in response to data growth.

## Scaling Static Content

The web layer in the architecture primarily handles data display, collection, and initial processing before passing it to the application layer. In the context of an e-commerce website, this layer deals with a significant amount of static content, like images and videos for products, creating a read-heavy workload. This content poses challenges related to storage, scalability, and load latency.

Storing large amounts of static content on web servers consumes substantial storage space. As the number of product listings grows, ensuring storage scalability becomes crucial. Moreover, high-resolution images and videos have large file sizes, potentially causing delays for users. A solution to address this issue involves leveraging a Content Distribution Network (CDN) to implement content caching at edge locations.

CDN providers like Amazon CloudFront, Microsoft Azure CDN, and Google CDN offer edge locations worldwide. These locations cache static content, such as images and videos, closer to users, reducing latency and improving content delivery speed.

For effective scaling of static content storage, the recommendation is to use object storage solutions like Amazon S3, or an on-premise custom origin. These solutions enable independent growth of storage capacity, decoupled from server memory and computing capabilities. Utilizing popular object storage services like Amazon S3 also yields cost savings. These storage options can even store static HTML pages, alleviating the load on web servers and enhancing user experiences by reducing latency through CDN utilization.

## Scaling Server fleet

The application tier plays a vital role in processing user requests from the web tier and handling complex business logic interactions with the database. As user requests surge, the application tier must scale to meet the demand, then scale down as demand decreases. However, scaling without addressing user sessions can lead to a poor user experience, as it may disrupt their ongoing activities.

To tackle this, the initial step is to manage user sessions independently from the application server instances. This involves storing user sessions in a separate layer, often utilizing NoSQL databases. These databases, designed as key-value stores, excel at handling semi-structured data. This is particularly useful since different users may input varying attributes, making traditional relational databases less suitable due to rigid schemas. For instance, one user might provide just a name and address, while another includes additional details like phone number, gender, and marital status. NoSQL databases like Amazon DynamoDB shine here, being partitionable and capable of horizontal scaling to a degree that other databases can't achieve.

By storing user sessions in NoSQL databases such as Amazon DynamoDB or MongoDB, horizontal scaling of application instances becomes feasible without impacting the user experience. With the addition of a load balancer in front of a fleet of application servers, the workload can be efficiently distributed among instances. The use of auto-scaling further enhances this setup, allowing instances to be automatically added or removed based on demand fluctuations. This approach ensures that users' ongoing sessions remain intact even as the application infrastructure scales dynamically.

## Scaling Database

Scaling relational databases horizontally can be complex due to the need for techniques like sharding, which involves modifying the application significantly. However, preventive measures can be taken to reduce the load on the master database. Combining storage methods such as utilizing separate NoSQL databases for user sessions, employing object stores for static content, and implementing external caching helps alleviate the burden on the master database.

A strategy involves using a read replica to handle read requests while keeping the master node for writes and updates. Amazon RDS supports up to six read replicas, and tools like Oracle plugins enable real-time data synchronization. Caching engines like Memcached or Redis can further ease the load on the master node by storing frequent queries.

In the event that the database grows beyond its capacity, the solution might involve redesigning and partitioning the database into shards. Each shard can grow independently, with user data assigned to specific shards based on a chosen partition key (e.g., user_name). This key determines which partition stores a user's data based on the first letter of their name.

Scalability is a pivotal consideration in architecture design, impacting budget and user experience. Solution architects must prioritize elasticity, optimizing workloads for performance and cost-effectiveness. Evaluating options like CDNs for static content scaling, autoscaling for server load management, and various data storage choices (caching, object stores, NoSQL, read replicas, and sharding) is crucial.

The article emphasizes the importance of injecting elasticity into different layers of architecture. This ensures high application availability and resilience, ultimately contributing to a robust application.

---
