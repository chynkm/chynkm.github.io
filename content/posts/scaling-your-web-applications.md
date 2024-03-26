---
author: "Karthik M"
title: "System design 101: Scaling your web applications"
date: "2020-11-27"
canonicalUrl: https://litebreeze.com/blog/2020/11/27/scaling-your-web-applications/
tags:
- system design
- server
- database
- aws
---

When I started programming web applications, I focused on a very small userbase. As I gained experience, the application
userbase expanded too. In the process, I had to unlearn many concepts as well. I was forced to think of high
availability, zero downtime deployments, and a large  number of database read/writes etc. These lessons were learnt the
hard way as I faced computational complexity – for example, when an app didn’t scale according to the usage, or when
assets couldn’t be served over CDN, and more..

As in the world of computer programming and system design, most of these problems were already encountered by my peers
in the same domain. The first such concept, which piqued my interest was the
[The Twelve-Factor App](https://12factor.net/). Here, I understood the reasons and the indispensable need for separating
concerns and entities.

The following is a very high level overview of my understanding of system design and architecture.

Most basic web applications do one thing and do it well: Create, Read, Update and Delete information on a relational
database. When the user count increases, the database interactions increase. For a normal web application, the first
point of bottleneck is the database.

## First system
In a simple web application, components are all hosted on the same server i.e. the web server, the database and the
application assets all stored on the same server. When the user count increases, the server response time increases
accordingly. The easiest solution is to add more hardware resources [RAM (memory), CPU (processing power)] by buying
even bigger, faster, and more expensive machines. This is termed as Vertical Scaling. This architecture will provide
performance improvement upto a certain amount of additional users. Once this threshold is breached, there isn’t much to
be done on the existing server.

<img src="/images/first-system.png" alt="First system" width="100%">

## Second system
The next step would be to host the database on a separate machine and the web application on another. Again, this design
only supports a number of users upto a certain threshold.

<img src="/images/second-system.png" alt="Second system" width="100%">

## Third system
Once we surpass the hardware limit for a single machine, the only way to handle the application is by adding more
machines in a cluster. Adding more machines to the existing architecture is called Horizontal Scaling.

My goto cloud provider is AWS. I accomplish horizontal scaling of servers using Amazon EC2/EKS/ECS/Fargate with auto-scaling
configurations behind an ELB(load balancer). The auto-scaling configuration deals in provisioning new machines when the
user requests increases and removes them upon decreasing. Custom AMIs (Amazon Machine Images) created using
[Packer](https://www.packer.io/) or Docker containers are used for provisioning new machines.

The new machine’s application codebase is either cloned from a repository or a Docker image is pulled and started inside
it or a Docker container is directly run on the infrastructure. The load balancer evenly distributes requests from the
users onto the application cluster. Applications in the machines are updated by executing rolling restarts or by
updating the Docker image versions.

**When the building blocks are too simple, the complexity moves into the interaction between the blocks.**

This architecture presents a developer with a lot of questions. In the initial architecture, all code and user related
data like profile pictures (assets) and login information (sessions) were stored on a single machine.

When machine count increases, the storage cannot reside on multiple machines. Information residing on multiple machines
will provide inconsistent information. Moreover, machines behind a load balancer are spinned-up and dropped depending on
user traffic/performance. Hence, the storage needs to be external from the horizontally scaled machines.

For assets, we use a centralised data store, which is accessible from all the scaled machines eg:
[Amazon S3](https://aws.amazon.com/s3/), [DO Spaces](https://docs.digitalocean.com/products/spaces/). For sessions, we
utilise an external database/in-memory cache called Redis eg: [Amazon ElastiCache](https://aws.amazon.com/elasticache/).

As we horizontally scale the application servers, the next bottleneck is the database.
1. Option one is to keep the RDBMS running. Employ master-master replication or master-slave replication, read from one
database and write to another database etc. Managing/tuning a database will require a lot of effort.
2. Option two is to use managed services like [Amazon Aurora Serverless](https://aws.amazon.com/rds/aurora/serverless/).
When the traffic increases, the database will automatically scale to handle them.
3. The third option is to use a NoSQL datastore. These datastore will scale horizontally just like the application
servers. The downside is that relational elements are lost and JOINs are to be handled on the application code. eg:
CouchDB, MongoDB, Cassandra etc.

After a certain point, even the datastore reaches its limitation. Caches are values stored in memory (RAM) as a
key-value structure. RAMs provide a lightning-fast response. Databases are battery included with caches. Tweaking these
values according to the data/queries will provide better performance. When it comes to web applications, the data
retrieval should first check the application’s memory cache. If the cache doesn’t hold value, it should query the
database. This value is then updated in the cache for subsequent fetches. eg: Redis, Memcached etc.

A major delay for a web application happens when it interacts with third party services or executes a compute intensive
task. An easy example is the verification of an email address as a new user completes his registration. The web page
waits for the response form the third party service until the email sending process is completed. This delay downgrades
the user experience. A better way to handle them would be to push the task to a queue, which are then picked by other
processes and completes the task.

In this manner, data processing won’t act as a bottleneck if there is an unexpected spike in usage. Moreover, data in
the queue isn’t lost and can work depending on the available resources. This asynchronous processing improves the
application performance for time consuming tasks. RabbitMQ, Redis, Amazon SES are a few queue management softwares.

CDN (Content Delivery Network/Content Distribution Network) is a geographically distributed network of proxy servers and
their data centers. A CDN allows for the quick transfer of (primarily static) assets needed for the web application like
HTML pages, JavaScript & CSS files, multimedia content like images/videos etc. The number of requests to the web server
is decreased as the assets are loaded from the CDN. Leveraging a CDN will speed up website delivery to the users and
decrease the impact/load on the webserver.

Incorporating all the above features will result in the following architecture.

<img src="/images/third-system.png" alt="Third system" width="100%">

The approach to system design depends totally on the web application requirement and its users. Common questions that
will provide the outline of the system architecture are:

1. What kind of traffic are we expecting i.e. how many users does the system need to handle in a time period?
2. How much data needs to be handled and the expected read-to-write ratio?
3. How many requests per second do we expect for the API?

The above example isn’t a silver bullet. Each architecture needs to be studied differently and developed according to
its requirements and use-cases.
