# AWS Foundations — Serverless, Monoliths, Lambda and EC2 | 9 Oct 2026

> **Study record — 9 October 2026.** These notes consolidate the two AWS videos assigned in the morning learning slot, the AWS console concepts encountered during account setup, and supporting explanations checked against AWS documentation. The video transcript service was unavailable, so this is a structured, documentation-verified study guide to the subjects of the videos rather than a word-for-word reconstruction of everything said by the instructor. Practical EC2 deployment and SSH are reserved for a later lesson.

**Videos studied**
- [Piyush Garg — What is Serverless? | Serverless Vs Monolith | AWS Lambda](https://www.youtube.com/watch?v=AgOmeANl3ls)
- [Piyush Garg — Amazon EC2: Elastic Cloud Server & Hosting with AWS](https://www.youtube.com/watch?v=-FKQwXtrSSQ)

<callout icon="💡" color="blue_bg">
	The central idea is not that one AWS service is universally better. A developer is choosing how much infrastructure to manage, how an application is packaged, and how its demand changes over time. These are different design decisions, and understanding their separation makes the rest of AWS much easier to reason about.
</callout>

## Part I — Why cloud computing changes how we build software

### 1. A web application needs a computer somewhere

When a developer runs a Node.js application on a laptop, the operating system allocates memory, schedules the process on the CPU, and allows the application to listen on a network port. A production application needs the same fundamental resources, but it must remain accessible when the developer closes the laptop. It also needs reliable networking, appropriate security, storage, and a way to cope with changing traffic.

Before cloud platforms became commonplace, an organization often had to buy or rent physical servers and arrange their installation, maintenance, power, networking, and replacement. This required money upfront and involved guessing how much capacity the application would need months in advance. Underestimating demand could make a website unavailable; overestimating it could leave expensive hardware sitting idle.

Cloud computing replaces much of this upfront commitment with resources that can be provisioned on demand. AWS owns and operates the underlying infrastructure while customers use services such as EC2 and Lambda to run their software. The customer still makes important decisions about security, configuration, architecture, and cost. The cloud does not eliminate engineering responsibility; it changes which parts are handled by the provider.

### 2. Three ideas that must not be confused

**Cloud computing** describes a way of obtaining computing resources as services rather than owning all the physical hardware yourself. Both a long-running virtual machine and an event-driven function can operate in the cloud.

**Application architecture** describes how the application's functionality is divided and deployed. A monolith groups significant parts of the application into one deployable unit, whereas a distributed design may deploy multiple independently operating components.

**Execution or hosting model** describes how the application receives compute capacity and how much of that environment the developer manages. EC2 gives the customer an operating-system-level virtual server. Lambda normally lets the customer submit code that AWS runs in response to invocations.

These dimensions overlap, but they are not interchangeable. A monolithic backend can run on EC2, and a cohesive or even monolithic codebase can be adapted to Lambda. Conversely, multiple microservices may run on EC2. The useful question is not simply “monolith or serverless?” but “how should the code be organized, and who should operate the environment in which it runs?”

## Part II — Monolithic applications and serverless computing

### 3. What a monolithic application actually is

A **monolith** is an application whose major features are packaged and deployed together. Consider an online practice platform with authentication, test generation, question delivery, scoring, and user analytics. A monolithic backend could expose routes for all those features through one Node.js application and deploy them as one running service.

This structure can be a sensible starting point. Modules can share ordinary function calls, database transactions can be easier to coordinate, and local development is relatively straightforward. A monolith is not inherently poor engineering. A well-designed monolith can have clear module boundaries, tests, and separation of concerns without being split into separate network services.

The difficulty appears when the system becomes large enough that the advantages of a single deployable unit start turning into constraints. Changing a small feature may require redeploying the whole application. One part of the workload may need substantially more resources than another, yet scaling often means replicating the entire application. If internal module boundaries are weak, changes can have unexpected effects across the codebase.

A monolith can still scale horizontally by running multiple copies behind a load balancer. The distinction is that the replicas normally contain the same larger application, even when only one feature is under heavy load.

### 4. What “serverless” means

**Serverless does not mean that software runs without servers.** Physical machines still execute the code. It means that the provider takes responsibility for provisioning and operating more of the underlying compute infrastructure. Instead of preparing and managing a long-lived virtual machine, the developer supplies application logic and configures how it should be invoked.

AWS Lambda is a central example. A developer writes a function handler and connects it to an event source. An HTTP request routed through API Gateway, an object uploaded to S3, or a message from SQS can cause Lambda to execute that handler. AWS manages the execution environments and scales the capacity within the service's quotas and limits.

This model is often described as **event-driven** because the work begins when something happens. An event can represent a web request, a file upload, a scheduled action, or a queue message. The function receives information about the event, performs its task, and returns or records a result.

Serverless is not a synonym for microservices. A function is an execution unit; a microservice is an architectural boundary around a business capability. Creating hundreds of tiny Lambda functions does not automatically produce a well-designed system.

### 5. How Lambda executes a request

Imagine that an application must generate a thumbnail whenever a user uploads an image. One design is to keep an always-running server that repeatedly checks for new uploads. Another is to configure an S3 event to invoke a Lambda function when an object is created.

In the Lambda approach, the sequence is conceptually:

1. The application places an object in an S3 bucket.
2. An event integration supplies information about the new object to the function.
3. Lambda provides an execution environment and invokes the configured handler.
4. The handler retrieves the object, processes it, and stores the resulting output.
5. The invocation finishes; AWS controls whether the execution environment is retained for reuse.

This separation can reduce the need to maintain a server that does nothing during idle periods. However, the developer remains responsible for the application logic, permissions, error handling, and downstream costs of services used by the function.

### 6. Automatic scaling and the idea of concurrency

A traditional single server has finite CPU and memory. If incoming requests arrive faster than the application can process them, response times increase, requests queue up, or the service fails. Scaling may require adding more machines and routing requests between them.

For ordinary AWS Lambda functions, AWS can create additional execution environments when more invocations must be handled concurrently. **Concurrency** means the number of requests currently in progress, not the total number of requests received in a day. Automatic scaling is valuable, but it is not infinite: concurrency quotas, scaling rates, downstream database capacity, and service-specific limits still matter.

A system can also fail outside Lambda. For example, increasing the number of parallel function invocations might overwhelm a database connection limit. The compute layer may scale successfully while the overall application becomes less reliable. Scaling therefore needs to be considered end to end.

### 7. Cold starts, warm starts, and statelessness

When Lambda needs a new execution environment, it must initialize the runtime and application code before the handler can serve the invocation. This additional startup work is commonly called a **cold start**. A reused execution environment may complete later requests more quickly; this is commonly called a **warm start**.

Cold starts are an important trade-off for latency-sensitive applications, but they are not a reason to dismiss Lambda for every API. Their effect depends on language runtime, initialization work, deployment size, traffic patterns, and configuration. AWS also offers techniques such as provisioned concurrency when predictable startup latency is important.

Normal function invocations should be designed **as if their local state will not survive**. A reused environment may retain in-memory objects or temporary files, but that reuse is not guaranteed. Important user data, sessions, and durable task progress belong in an appropriate external datastore or storage service. This is why serverless application design often pairs functions with managed databases, queues, or object storage.

### 8. When serverless is useful—and when it is not

Lambda works especially well for short-lived, event-driven tasks, intermittent workloads, scheduled automation, file processing, and APIs where avoiding day-to-day server administration is valuable. A function that runs only when a new file arrives is a natural fit: its operating pattern already matches an event.

A continuously busy backend, a workload requiring unusually long computations, software dependent on extensive operating-system control, or an application with strict and consistent latency requirements may need a different model. Serverless pricing also depends on usage. An architecture made of many separately billed services can become harder to predict and operate if the design is unnecessarily fragmented.

For the standard Lambda function model, an individual invocation has a time limit, so it should not be treated as a permanent server process. Modern AWS Lambda offers additional compute and workflow features, but those should be learned separately rather than assumed to behave exactly like a conventional function invocation.

The sound engineering decision is to compare workload duration, frequency, latency, operational effort, scaling needs, and total cost. “Serverless is modern” and “monoliths are outdated” are not sufficient reasons to choose an architecture.

## Part III — Amazon EC2 and the virtual-server model

### 9. What EC2 gives a developer

**Amazon Elastic Compute Cloud (EC2)** provides configurable virtual servers called *instances*. An instance behaves in many ways like a computer that you rent: it has an operating system, a hardware capacity profile, networking, and storage. You can install a runtime, start application processes, configure a web server, and decide when to stop or terminate it.

This is useful when you want control over the operating system and the application environment. A Node.js backend could run on an Ubuntu EC2 instance as a long-lived process, much as it runs locally, except that it is hosted in an AWS data center.

The word **elastic** refers to the ability to change capacity as needs change. You can choose a different instance type or add more instances instead of purchasing new physical hardware. This flexibility does not mean every scaling operation is instantaneous or automatic; you must configure the appropriate process and services.

### 10. The choices involved in launching an instance

The EC2 launch page asks several questions because each one defines a different part of the server.

**Amazon Machine Image (AMI).** The AMI is the template from which the instance is created. It determines the operating-system image and can include preconfigured software. Choosing Ubuntu or Amazon Linux is analogous to deciding what operating system should already be installed when the computer first starts. The AMI is not the hardware size.

**Instance type.** The instance type defines the available compute profile, including CPU, RAM, and related performance characteristics. A small general-purpose instance may be sufficient for experimentation, whereas a memory-intensive application or compute-heavy workload could require a different family. The label does not by itself guarantee that usage is free; eligibility and costs must be checked for the specific account, region, and configuration.

**Key pair and access.** A key pair can be used to authenticate when connecting to a Linux instance over SSH. The public key is registered for the instance, while the private key must remain private. Anyone who obtains the private key and has the required network access may be able to authenticate as the corresponding user.

**Network configuration.** An instance is launched inside a Virtual Private Cloud (VPC), which provides a logically isolated networking environment. Subnets, routing, and public IP assignment influence whether the instance can be reached from the internet. A public IP does not by itself make every port reachable.

**Security group.** A security group acts as a virtual firewall around supported AWS network interfaces. Its inbound rules specify which traffic can reach the instance, and its outbound rules control permitted outgoing traffic. For example, SSH uses TCP port 22; it is safer to allow that port from your own IP address rather than the entire internet. A website may require HTTP or HTTPS traffic on ports 80 or 443, depending on its setup.

**Storage.** The root disk often uses Amazon Elastic Block Store (EBS), which provides persistent block storage. Some instance types also expose instance store volumes, whose data is temporary and tied to the host lifecycle. This distinction becomes important when an instance is stopped, replaced, or terminated.

Before clicking *Launch*, you should be able to explain the AMI, instance type, key pair, security group, networking, and storage selections. Making those choices intentionally matters more than memorizing the sequence of buttons.

### 11. Public IP addresses and SSH

A server needs an address if a client or administrator is going to reach it over the network. Within a VPC, a **private IP address** supports internal communication. A **public IP address**, together with the required network route and security rules, can make internet communication possible.

SSH (Secure Shell) is a protocol for securely operating a remote machine through a terminal. With a Linux EC2 instance, you typically connect using the correct username, instance address, and authentication key. Different AMIs can use different default usernames.

A normal automatically assigned public IPv4 address can change when an instance is stopped and started again. An Elastic IP is designed to provide a static public IPv4 address, but public IPv4 addressing may incur additional charges. Neither an Elastic IP nor a public IP should be added without a reason.

The later SSH lesson will show the actual connection procedure. At this stage, the important understanding is that **compute**, **network reachability**, **firewall permissions**, and **authentication** are four separate requirements. If an SSH connection fails, the cause could be in any of those areas.

### 12. Instance states and what happens to data

An EC2 instance moves through states such as *pending*, *running*, *stopping*, *stopped*, *shutting-down*, and *terminated*. These states are not interchangeable.

**Stopping** an EBS-backed instance usually halts the virtual machine while preserving its EBS volumes. Instance compute charges normally cease after the relevant stopping transition, but attached EBS storage can continue to cost money. A stopped instance is therefore not automatically a zero-cost resource.

**Starting** the stopped instance makes it run again. Its private IP can remain the same, but a normal automatically assigned public IPv4 address can change. Temporary instance-store data is not preserved across a stop/start.

**Terminating** an instance is different from stopping it. Termination permanently ends that EC2 instance. Whether a particular EBS volume is deleted depends on its configured deletion behavior, so resource cleanup should be verified rather than assumed.

### 13. What EC2 costs

EC2 generally follows a pay-for-resources-used model. For an On-Demand instance, the compute bill depends on its type, platform, region, and running time. Additional items may include EBS volumes, snapshots, data transfer, public IPv4 addresses, or other connected AWS services.

AWS has multiple purchasing options, such as On-Demand, Savings Plans, Reserved Instances, and Spot capacity. On-Demand offers flexibility without a long-term capacity commitment. Savings Plans and reservations exchange some flexibility or commitment for potential discounts. Spot capacity can cost less, but it can be interrupted and is therefore suited to workloads that tolerate interruption.

**Free Tier is not the same as “everything on AWS is free.”** Account plans, credits, eligible resources, geographical region, quotas, and current promotional rules determine what is covered. Before a hands-on exercise, check the actual billing page and applicable pricing rather than trusting an older tutorial's “free tier eligible” label.

## Part IV — EC2 versus Lambda: the decision in context

### 14. Two ways to host a Node.js backend

Suppose you are developing a web application that serves a few API endpoints. With **EC2**, you can launch an instance, install Node.js, deploy the application, and keep the process running. You manage operating-system updates, application restarts, security configuration, and capacity planning. This gives you considerable freedom, but also ongoing operational work.

With **Lambda**, you can expose functions through an HTTP integration or trigger them through events. You do not maintain a general-purpose operating-system server for those functions, and AWS manages much of the execution infrastructure. You must instead design around invocation boundaries, deployment configuration, service integration, state storage, and relevant function limits.

Neither approach dictates whether your business code is organized as one well-modularized application or several independently deployed services. The architecture of the code and the hosting of that code should be evaluated separately.

### 15. A practical decision rule

Use EC2 when you want an ordinary server environment, control over installed software and long-running processes, or a straightforward place to host an existing backend with explicit responsibility for operations.

Consider Lambda when the work naturally starts from events, may be idle for long periods, and can be expressed as bounded invocations without needing control over a persistent machine. It is particularly attractive when reduced infrastructure administration outweighs its execution and integration constraints.

As an application grows, hybrid choices are normal. The public API might run on long-lived compute, while an asynchronous thumbnail generator or scheduled cleanup operation runs in Lambda. An architecture is a collection of trade-offs, not a loyalty test between AWS services.

## Part V — What was actually explored in today's AWS console session

### 16. Account creation and the AWS Management Console

An AWS account was created and the AWS Management Console became accessible after the signup flow initially reported an error during billing verification. The console is a graphical way to discover and configure AWS services; it is not the same thing as the virtual servers or functions that ultimately execute application code.

AWS displays a selected **Region** for regional services. A Region is a geographical AWS location, such as Stockholm or Mumbai. Different Regions have different available resources and potentially different prices. IAM is a global service, which is why the IAM screens show *Global* instead of an ordinary deployment Region.

### 17. Root user, IAM, and the unfinished MFA setup

An account's **root user** is the highly privileged identity associated with the original account credentials. Because compromise of that identity can expose all account resources, AWS recommends protecting it with multi-factor authentication and using it only when root-level access is necessary.

**IAM** stands for Identity and Access Management. It is the AWS service for defining who can perform which operations on which resources. IAM permissions matter even when a resource exists and its network configuration is correct: a person can see a service in the console but still be denied permission to create, modify, or delete its resources.

The IAM user list displayed **zero users**, which is normal immediately after account creation. Root user and IAM user are different identity types. AWS generally recommends federated workforce access through **IAM Identity Center** rather than routinely operating as root or creating unnecessary long-lived IAM user credentials.

The **Assign MFA device** screen was opened, but MFA was intentionally deferred so the learning schedule could continue. That means MFA should be treated as **not yet confirmed enabled**, not as a completed security step. A passkey/security key or a trusted authenticator app can be used to add a second sign-in factor. Never save or share a private key, MFA seed QR code, one-time code, or account recovery secret in ordinary study notes.

### 18. Billing visibility and safeguards still to be checked

The console's Cost and Usage widget displayed *Unable to load*. That message does not establish that AWS usage is free, nor does it by itself prove that the account has a billing problem. The next sensible step is to confirm the account's plan, credits, and cost visibility directly in Billing and Cost Management.

Before launching chargeable resources, set an AWS Budget or other suitable cost alert and review the estimated cost of the selected EC2 configuration. Budget notifications are helpful warnings, but a conventional budget alert is not a universal hard spending limit. Cleanup remains essential after experiments.

### 19. Today's progress and the next lab

**Completed:** Both assigned introductory videos were watched, an AWS account was created, the Management Console was explored, the IAM users page was opened, and the root-user MFA setup page was reached.

**Not completed:** MFA enrollment, administrative daily-use identity setup, billing/budget confirmation, and a live EC2 instance launch were not confirmed. The EC2 launch and SSH walkthrough belong to the later hands-on lesson, where the instance should be cleaned up afterward.

This distinction matters because study notes should record understanding without implying that practical steps have been carried out when they have not.

## Part VI — Mental model to carry into later lessons

When a browser sends a request to an application, the application needs **compute** to execute code, **networking** to receive traffic, **identity and permissions** to authorize cloud actions, and often **storage** to persist data. EC2 and Lambda solve the compute problem at different levels of abstraction. VPCs and security groups deal with network access. IAM deals with permissions. EBS and services such as S3 deal with different storage needs.

A useful habit is to ask four questions for any new AWS service: *What problem does it solve? What does AWS manage? What do I still manage? What can generate a bill?* If you can answer those clearly, the console becomes a tool for implementing decisions rather than a set of unfamiliar buttons.

## Sources and reliable follow-up reading

- [AWS Lambda overview](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) — its standard function model and AWS-managed responsibilities.
- [Lambda execution-environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html) — cold starts, warm starts, and temporary execution environments.
- [Lambda scaling behavior](https://docs.aws.amazon.com/lambda/latest/dg/scaling-behavior.html) — concurrency and scaling limits.
- [Amazon EC2 documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/) — virtual instances and their configuration.
- [EC2 instance lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html) — stop, start, terminate, and storage/billing effects.
- [Create an EC2 security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-security-group.html) — controlling inbound and outbound traffic.
- [AWS IAM getting started](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html) — identities, permissions, and recommended account access.
- [Root-user MFA](https://docs.aws.amazon.com/IAM/latest/UserGuide/enable-mfa-for-root.html) — securing the original AWS account identity.
- [AWS Free Tier FAQs](https://aws.amazon.com/free/free-tier-faqs/) — current plan terms and credits; verify your own account rather than assuming eligibility.
