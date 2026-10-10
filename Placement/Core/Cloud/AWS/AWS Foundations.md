# AWS Foundations — Serverless, Monoliths, Lambda and EC2 | 9 Oct 2026

> **Study record — 9 October 2026.** These notes consolidate the two morning AWS videos, the evening SSH lesson, the AWS console concepts encountered during account setup, and supporting explanations checked against AWS documentation. The video transcript service was unavailable, so this is a structured, documentation-verified study guide to the subjects of the videos rather than a word-for-word reconstruction of everything said by the instructor. A live EC2 deployment and SSH login remain for a later hands-on lab, even though the SSH procedure has now been studied.

**Videos studied**
- [Piyush Garg — What is Serverless? | Serverless Vs Monolith | AWS Lambda](https://www.youtube.com/watch?v=AgOmeANl3ls)
- [Piyush Garg — Amazon EC2: Elastic Cloud Server & Hosting with AWS](https://www.youtube.com/watch?v=-FKQwXtrSSQ)
- [Piyush Garg — How to SSH into Amazon EC2 Machine | SSH AWS EC2](https://www.youtube.com/watch?v=57TCFZG08oM)

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

Part VII now explains the SSH connection procedure in detail; a real instance connection remains for the practical lab. At this stage, the important understanding is that **compute**, **network reachability**, **firewall permissions**, and **authentication** are four separate requirements. If an SSH connection fails, the cause could be in any of those areas.

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

**Completed:** All three assigned AWS videos were watched (serverless, EC2, and SSH), an AWS account was created, the Management Console was explored, the IAM users page was opened, and the root-user MFA setup page was reached.

**Not completed:** MFA enrollment, administrative daily-use identity setup, billing/budget confirmation, and a live EC2 instance launch were not confirmed. The actual EC2 launch and successful SSH login belong to a later hands-on lab, where the instance should be cleaned up afterward.

This distinction matters because study notes should record understanding without implying that practical steps have been carried out when they have not.

## Part VI — Mental model to carry into later lessons

When a browser sends a request to an application, the application needs **compute** to execute code, **networking** to receive traffic, **identity and permissions** to authorize cloud actions, and often **storage** to persist data. EC2 and Lambda solve the compute problem at different levels of abstraction. VPCs and security groups deal with network access. IAM deals with permissions. EBS and services such as S3 deal with different storage needs.

A useful habit is to ask four questions for any new AWS service: *What problem does it solve? What does AWS manage? What do I still manage? What can generate a bill?* If you can answer those clearly, the console becomes a tool for implementing decisions rather than a set of unfamiliar buttons.

## Part VII — Securely connecting to an EC2 Linux instance with SSH

> **Evening AWS lesson — 9 October 2026.** The video [How to SSH into Amazon EC2 Machine | SSH AWS EC2 — Piyush Garg](https://www.youtube.com/watch?v=57TCFZG08oM) was completed. These notes explain the connection procedure and its security model using AWS's current documentation. **The video was watched, but a live instance was not launched and no actual SSH login was performed.** MFA and billing safeguards remain pending. The video transcript was not available, so the explanation is a documentation-aligned study guide rather than a word-for-word retelling.

### 20. What SSH means, and why we need it

**SSH (Secure Shell)** is a protocol for establishing an encrypted, authenticated connection to another computer over a network. Its most familiar use is remote administration: a developer opens a terminal on a laptop, establishes an SSH connection to a Linux server, and runs commands as though the terminal were opened on that remote machine.

This distinction is fundamental. Before connecting, the terminal is running commands on the developer's own computer. After successfully logging in, the shell is running **on the EC2 instance**. If you type `pwd` or inspect an application log after connecting, you are looking at the remote server's working directory or files, not those of the laptop.

SSH involves a **client**, typically the OpenSSH program on your laptop, and an **SSH server**, generally `sshd` on the remote Linux machine. The client and server negotiate an encrypted session, the client establishes which server it is contacting, and the server checks whether the user is allowed to log in.

A deployed Node.js backend provides a concrete example. SSH may be used to inspect its configuration, read logs, restart a process, or troubleshoot a failed deployment. SSH does not itself deploy the application, create an EC2 instance, or stop AWS charges; it is the secure access mechanism through which an administrator can operate an existing server.

### 21. SSH key pairs: proving identity without sending a password

For the common EC2 Linux SSH workflow, authentication uses a **public/private key pair**. The **public key** is registered with the server for the intended Linux user. The **private key** stays on the developer's laptop. When connecting, the SSH client proves possession of the corresponding private key without transmitting the private key itself.

A typical EC2 launch lets you select or create a key pair. If AWS generates it, you download the private-key file, commonly ending in `.pem`, when the key pair is created. You should not assume that the same private key can be downloaded again later. Losing it can make the standard login method unavailable, requiring a separate recovery procedure.

Think of the two keys as complementary responsibilities: the remote machine knows **which public key is permitted**, while the laptop retains the **private proof needed to authenticate**. The public key can be shared for its intended purpose. The private key must not be copied into GitHub, chat messages, screenshots, or study notes.

**Do not confuse AWS identity with Linux identity.** The AWS root user or an IAM identity may have permissions to create, inspect, or terminate EC2 resources through AWS APIs. SSH authentication, however, determines access to an **operating-system user account inside the instance**. Access to the AWS console is not automatically permission to SSH into a server, and the username for SSH is usually not `root`.

### 22. The requirements for a successful SSH connection

Four different parts of the system must be correct before the SSH command can succeed.

**The instance must be ready.** It should be in the running state, have passed its relevant status checks, and have an SSH server configured for the selected access method. An instance still starting up might be visible in the AWS console before it can accept an SSH connection.

**There must be a reachable network path.** For the simplest connection from a home laptop over the public internet, the EC2 instance needs an appropriate public IPv4 address or DNS name, and the VPC/subnet must permit that internet route. An instance's private IP address is normally not directly reachable from an unrelated home network. A public IP by itself does not guarantee that a network route exists.

**The security group must permit the connection.** A security group is an AWS virtual firewall associated with the instance's network interface. Standard SSH uses **TCP port 22**. For a direct connection from your laptop, a suitable inbound rule permits SSH on port 22 from **your current public IP address**, often selected using *My IP* in the console. Permitting SSH from `0.0.0.0/0` instead makes the port reachable from any IPv4 address and is unnecessary for an ordinary beginner lab.

**The client must authenticate as the correct user.** The remote Linux account, the key that was authorized when the instance was configured, and the key file supplied in the SSH command must match. The default username depends on the **AMI (Amazon Machine Image)**. Common Amazon Linux images use `ec2-user`, while common Ubuntu images use `ubuntu`.

These conditions form a useful diagnostic sequence: **Is the instance healthy? Can my network reach it? Is port 22 allowed? Am I using the correct account and key?** A correct key cannot repair a blocked port, and an open port cannot make an incorrect key authenticate.

### 23. Understanding the SSH command

On an Arch Linux laptop with an OpenSSH client, a conventional example is:

```bash
ssh -i ~/.ssh/aws-demo.pem ec2-user@203.0.113.10
```

This is an **illustrative command**. The IP address `203.0.113.10` is reserved for documentation, not the address of a real lab instance. The eventual command must use the instance's actual address and the key-file path on your laptop.

The first part, **`ssh`**, starts the local SSH client. The option **`-i ~/.ssh/aws-demo.pem`** specifies the private-key file, also called an *identity file*, to use for authentication. The expression **`ec2-user@203.0.113.10`** identifies the user account and remote server. The `@` separates the operating-system username from the network address.

For an Ubuntu AMI, the same command would typically begin with **`ubuntu@`** instead of `ec2-user@`. The username is not selected according to your personal name or AWS account: it is determined by the operating-system image or by an account that has been explicitly created on that instance.

Before a future lab, `ssh -V` can confirm that the **local** SSH client is installed. It checks a program on your laptop only; it does not create AWS resources or connect to a server.

### 24. Private-key permissions and the server's host key

Because an SSH private key is effectively a credential, the client expects its file to be kept private. On Linux, a key that is readable by unrelated users can trigger an **unprotected private key file** warning and be rejected.

A common way to restrict the file is:

```bash
chmod 400 ~/.ssh/aws-demo.pem
```

The command `chmod` changes filesystem permissions. The value `400` grants the file's owner read access while withholding access from other users. The setting `600`, which allows the owner both read and write access, is also restrictive. What matters is that the private key is **not accessible to other local users**.

Notice that this path refers to a file **on the laptop**. One common mistake is to type a key filename that exists in a different directory, or to mistake the name of an EC2 key pair in the AWS console for the exact path where its private file was saved.

SSH also authenticates the **server**. When contacting a host for the first time, the client may display a **host-key fingerprint** and ask you to confirm it. This *server host key* is not the same as your *user login key*. Checking the fingerprint against a trusted source helps avoid connecting to an unexpected machine. A later warning that the host's identity has changed deserves investigation rather than an automatic bypass.

### 25. What to do after a successful login

After connecting, three harmless commands provide a useful orientation:

```bash
whoami
hostname
pwd
```

`whoami` displays the Linux user running commands. `hostname` displays the current machine's hostname, and `pwd` prints its working directory. They help confirm both **which server** was reached and **which account** is active.

When finished, type `exit` to leave the remote shell and return to your laptop's terminal. **Exiting SSH does not stop the EC2 instance.** The instance may continue running, serving traffic, and incurring charges. Terminating or stopping an instance is a separate action carried out through AWS controls or APIs.

This distinction is especially important for short experiments: a closed terminal is not evidence that the cloud resource was deleted.

### 26. How to reason about common errors

**`Permission denied (publickey)`** usually means the remote SSH service was reached but did not accept the attempted key authentication. Common causes include the wrong `.pem` file, the wrong default Linux username, or a key pair that does not match the instance. Check those details before changing unrelated network settings.

**`UNPROTECTED PRIVATE KEY FILE`** indicates that the SSH client considers the local private-key file insufficiently protected. Use appropriately restrictive local permissions, such as `chmod 400`, and confirm that you are modifying the correct file.

**`Connection timed out`** points toward a reachability issue. Check the instance's status, the public IP or DNS name, the subnet route, TCP port 22 in the security group, and whether your local network blocks outgoing SSH. Repeatedly changing the key will not solve a network timeout.

**`Connection refused`** typically indicates that the destination was reachable enough to reject the connection but was not accepting SSH on the requested port. The SSH server may be stopped, configured differently, or actively rejected by networking rules.

**A warning about changed host identification** is different from a key-file-permissions error. It concerns the identity of the remote SSH server. A new instance or changed host may explain it, but the safer response is to verify the new identity before altering the local `known_hosts` entry.

If the cause remains unclear, the `-v` option prints SSH diagnostic information. This can help distinguish failures during network connection, host verification, and user authentication; any logs shared for help should first be checked for sensitive details.

### 27. SSH from a terminal versus EC2 Instance Connect

AWS also offers **EC2 Instance Connect**, which can provide browser-based access from the EC2 console or an integrated client flow. For supported images and configurations, it works with IAM authorization and provides a short-lived public key to the instance, avoiding the need to distribute the same long-lived downloadable key for every session.

The details matter. Depending on the chosen connection method, EC2 Instance Connect still needs compatible instance software, IAM authorization, and appropriate network/security-group access. A console connection can require SSH traffic from **AWS's EC2 Instance Connect service ranges**, rather than the public IP of your laptop. The **EC2 Instance Connect Endpoint** offers another pattern for reaching some private-address instances.

A further option, **Systems Manager Session Manager**, can permit remote administration without exposing inbound SSH, provided its agent, IAM permissions, and networking prerequisites are configured. These alternatives should not distract from the essential SSH lesson: understand which machine you are reaching, how the remote Linux user authenticates, and what network path carries the connection.

### 28. Next practical lab: what is planned and what remains undone

The objective for a later hands-on lab is deliberately small: **secure the account, create one disposable Linux instance, establish an SSH connection, run three harmless commands, and remove the resources deliberately**.

Before launching anything, finish **root-account MFA**, choose appropriate non-root daily-use access, and confirm billing visibility, available credits or free-plan eligibility, the selected instance's current price, public IPv4 charges, and a suitable budget or cost alert. A normal AWS Budget is an alert mechanism, **not a guaranteed hard cap** on spending.

Once those safeguards are confirmed, the practical steps will be to select one AWS Region; choose a Linux AMI and suitably priced instance type; create/download a key pair; configure a public network path and **TCP 22 from My IP** for direct SSH; wait until the instance is ready; restrict the private key's permissions; and connect using the right username. The verification commands are `whoami`, `hostname`, and `pwd`, followed by `exit`.

For cleanup, **terminate** the disposable instance once no data is needed. Check the relevant EBS volumes, snapshots, allocated public IP addresses, and any other resources that may persist. Stopping an instance is not the same as terminating it, and associated storage or public IPv4 allocations may still generate charges. The exact current console steps should be verified against the AWS EC2 getting-started documentation when the lab is actually performed.

**Progress as of tonight:** Three AWS videos have been completed. No successful SSH login, EC2 creation, account MFA enrollment, or billing/budget verification has been confirmed. Studying the procedure and performing the procedure are separate milestones.

### 29. Questions worth being able to answer

1. Why does knowing the AWS console password not prove that you can log in as a Linux user through SSH?
2. What is the difference between an **EC2 public IP address** and an **SSH public key**?
3. If the SSH connection times out, why should you check routing and security groups before looking for a new PEM file?
4. Why can `ec2-user` be correct for one Linux AMI while `ubuntu` is correct for another?
5. What changes when you type `exit`, and what does **not** change about the EC2 instance or its billing?
6. Why is opening TCP 22 only from your IP preferable to allowing inbound SSH from the whole internet?

**References for this lesson**

- [AWS — Connect to a Linux instance using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-to-linux-instance.html)
- [AWS — Connect using a local SSH client](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)
- [AWS — Default Linux usernames](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html)
- [AWS — EC2 Instance Connect prerequisites](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-prerequisites.html)
- [AWS — EC2 instance lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)

---

## Part VIII — Amazon S3, Node.js, and presigned downloads

> **Morning lesson — 10 October 2026.** This part accompanies Piyush Garg's [AWS S3 Simple Storage Service](https://www.youtube.com/watch?v=d8A8JmAImc4) and [How to Use AWS S3 with NodeJS?](https://www.youtube.com/watch?v=DOUxRYi2Fwg), both completed today. The explanations are based on the studied topics, the Node.js exercise and errors actually encountered, and the current AWS documentation. They are **not** a claim to reproduce the videos' complete transcripts. The separate evening lesson on **presigned uploads (PUT)** remains scheduled for later and is not marked as studied here.

### 30. Why object storage exists: storing files is different from running applications

Suppose a website allows students to upload question-bank PDFs, profile photographs, or recorded explanations. The application server needs to **receive instructions and execute logic**, but the files themselves may need to remain available long after any one server has been restarted or replaced. Saving every upload to a local disk attached to an application instance is a fragile design: another server might not see the file, deployment could replace the instance, and storing larger media could gradually exhaust the machine's capacity.

**Amazon Simple Storage Service (Amazon S3)** solves a different problem from EC2 and Lambda. S3 is AWS's managed **object-storage service**. It stores and retrieves data as objects, accessible through service APIs, rather than supplying an operating system in which application code runs. EC2 offers virtual machines; Lambda executes bounded pieces of code; S3 retains files and related metadata. In a practical web application, all three might participate in the same workflow without competing to do the same job.

The essential design idea is to separate **application execution** from **durable file storage**. A backend can keep file metadata—such as ownership, original filename, and who can access it—in a database, while S3 holds the actual bytes. This division also allows file delivery and retention policies to evolve independently of the machines that serve application requests.

### 31. Buckets, objects, keys, and the illusion of folders

The basic S3 storage container is a **bucket**. A bucket has a name and is created in a particular AWS Region. Inside a general-purpose bucket, each stored item is an **object**: the object's content plus associated information such as its key and metadata.

The **object key** is the name by which that object is identified inside its bucket. In ordinary object storage, the pair **bucket + key** locates the object. For example, a bucket named `study-assets-demo` may contain an object with the key `papers/2026/sample.pdf`. The full key includes the apparent path prefix. Asking S3 for `sample.pdf` is not equivalent to asking for `papers/2026/sample.pdf`; the latter is a different key.

A key can contain slashes, so the AWS console often displays familiar **folders**. In a general-purpose S3 bucket, those visual folders are usually **prefixes within keys**, not actual nested directories with their own filesystem semantics. A key such as `users/42/avatar.png` can be convenient for grouping objects without implying that S3 is an ordinary Linux filesystem.

S3 also lets an object carry **metadata**, including properties such as content type. A document served with `Content-Type: text/html` may be displayed as HTML by a browser; the same bytes served under another type may be handled differently. The extension at the end of a key does not, by itself, guarantee how the response is interpreted.

**Common mistake:** A beginner sees the console's folder layout and assumes that `GetObject` accepts a local computer path or automatically searches subfolders. It does neither. The request needs the exact object key inside the chosen bucket.

### 32. S3 Regions, endpoints, and the difference between an address and permission

A bucket is associated with an AWS **Region**, such as **Europe (Stockholm), `eu-north-1`**, or **Asia Pacific (Mumbai), `ap-south-1`**. Requests should be sent to the appropriate regional endpoint and signed using the bucket's actual Region.

For an ordinary virtual-hosted-style bucket request, a regional endpoint has a shape such as:

```text
https://BUCKET_NAME.s3.eu-north-1.amazonaws.com/OBJECT_KEY
```

This pattern identifies **where** the request is directed; it does not establish that the browser is **allowed** to read the object. A URL can be syntactically correct and still lead to an authorization error because storage location and access control are separate questions.

In today's Node.js exercise, the S3 client was initially configured for `ap-south-1`, while the bucket's actual endpoint identified `eu-north-1`. Opening the generated URL produced **`PermanentRedirect`**, along with an instruction to use the specified endpoint. The fix was to configure the client for the bucket's real Region and **generate a fresh presigned URL**, not merely replace the hostname in an already signed URL.

Why does the old URL not simply work after a redirect? **AWS Signature Version 4** ties a signature to details of the request, including the host, Region, and credential scope. A URL prepared for one regional endpoint cannot safely be treated as a valid signature for an arbitrary second endpoint. A redirected request may need to be signed anew. The simplest choice, when the bucket Region is known, is to create the client in that Region from the beginning.

**Common mistake:** Assuming that the Region shown in the AWS console's top-right selector automatically matches the Region configured in a locally running Node.js process. The SDK client has its own configuration, and the bucket has its own Region.

### 33. What the Node.js AWS SDK does for us

A Node.js application communicates with S3 through authenticated API requests. Instead of constructing signatures and HTTP requests manually, we can use **AWS SDK for JavaScript v3**. Its modular packages let an application import the clients and commands it needs without treating every AWS service as one enormous library.

For the read-only exercise, the two relevant packages are **`@aws-sdk/client-s3`** and **`@aws-sdk/s3-request-presigner`**. The first contains the S3 client and service commands. The second contains utilities that calculate a time-limited signature and place it in a request URL.

Three pieces of code have distinct responsibilities:

**`S3Client`** is configured with a Region and resolves the appropriate AWS credentials. It represents how the application will communicate with S3; constructing the client alone does not download an object.

**`GetObjectCommand`** describes the requested operation—retrieve an object using a particular **`Bucket`** and **`Key`**. Constructing this command is only a declaration of intent, not proof that the object exists or that the caller is permitted to read it.

**`getSignedUrl(client, command, { expiresIn })`** prepares a URL that authorizes the described operation for a limited period using the signing identity's credentials. For this example, generating the URL is different from **calling** `s3Client.send(command)` to fetch an object's bytes inside Node.js. The browser or another HTTP client performs the actual request later using the URL.

This separation is valuable because a backend can create a short-lived route to an object without becoming a proxy that downloads every file itself and streams the entire content to the user.

### 34. What a presigned GET URL actually represents

S3 buckets are often deliberately **private**. A private object normally cannot be retrieved through an ordinary unsigned URL by an anonymous visitor. A **presigned URL** solves a narrower problem: a trusted application signs a specific request so that a recipient can perform the authorized operation for a limited period, **without receiving AWS account credentials**.

For a presigned download, the operation is typically **`GetObject`**, corresponding to an HTTP **GET** request. The backend identifies the bucket and key, chooses a suitable expiration time, and uses authorized AWS credentials to sign the request. The recipient opens that signed URL in a browser, and S3 validates the signature before returning the object.

The query string commonly contains fields such as **`X-Amz-Algorithm`**, **`X-Amz-Credential`**, **`X-Amz-Date`**, **`X-Amz-Expires`**, **`X-Amz-SignedHeaders`**, and **`X-Amz-Signature`**. These fields collectively describe and authenticate the request. They are not decorations that can be edited independently. For example, changing `X-Amz-Expires` by hand does not extend access because the changed request would no longer match its signature.

A presigned URL is best treated as a **temporary bearer credential**: anyone who obtains a usable copy can potentially use it while it remains valid. It should not be committed to a repository, published in a public log, or shared with an unintended audience. It can often be used repeatedly before expiry rather than being automatically single-use.

The URL's effective lifetime cannot exceed the validity of the credentials that signed it. If an associated access key is deactivated, deleted, or expires, an otherwise unexpired link may stop working. Permissions can also change during its lifetime. The exact maximum duration depends on signing method and credential type; specifying `expiresIn` does not guarantee that the link will remain usable for that long.

**Important boundary:** A presigned URL grants access only as far as the **signer's existing permissions** allow. It cannot turn an IAM identity with no `s3:GetObject` privilege into one that is authorized to read private objects.

### 35. A safe and minimal Node.js example (SDK v3)

The following example recreates the **presigned GET concept**, not the later upload lesson. It avoids writing an AWS secret key directly into source code. The example assumes a Node.js project configured for ES modules, for example through `"type": "module"` in `package.json`.

Install the required packages in a separate learning directory:

```bash
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

Then create `index.js`:

```javascript
import { S3Client, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

const region = process.env.AWS_REGION;
const bucket = process.env.S3_BUCKET;
const key = process.env.S3_KEY;

if (!region || !bucket || !key) {
  throw new Error("Set AWS_REGION, S3_BUCKET and S3_KEY first.");
}

// The SDK resolves credentials from the configured AWS profile,
// environment, or workload role. No secret is stored in this file.
const s3 = new S3Client({ region });

const command = new GetObjectCommand({
  Bucket: bucket,
  Key: key,
});

const url = await getSignedUrl(s3, command, {
  expiresIn: 900, // 15 minutes, measured in seconds
});

console.log("Private S3 object — temporary GET URL:");
console.log(url);
```

This code is small enough that every line should be understood. **`region`** must be the Region of the bucket. **`bucket`** identifies the storage container. **`key`** identifies one exact object. The **`S3Client`** obtains the configured identity; the **`GetObjectCommand`** describes the read; and **`getSignedUrl`** creates the time-limited signed URL.

For a local development setup, configure an AWS profile using a supported secure login or credentials workflow, then supply the non-secret Region, bucket, and object-key settings. For example, with an already configured profile named `learning`:

```bash
AWS_PROFILE=learning AWS_REGION=eu-north-1 \
S3_BUCKET=my-private-demo-bucket \
S3_KEY=example.txt node index.js
```

The bucket and key above are **placeholders**, not objects guaranteed to exist in your account. The example also requires the selected AWS profile to be authorized to read that object. When running software inside AWS, a properly scoped **IAM role** with temporary credentials is generally preferable to a long-lived access key.

**Do not infer success from seeing a URL printed.** Signing the request shows that the client could construct a signature; it does not prove that S3 will authorize and return the object. A browser may still display `AccessDenied` or `NoSuchKey` when the link is actually used.

### 36. IAM permissions: why the signed link was denied

After the Region was corrected in today's exercise, the browser returned **`AccessDenied`** and named the IAM user **`nodejs`**. Crucially, the error explained that **no identity-based policy allowed `s3:GetObject`** for the requested S3 object.

This is an **authorization** failure, not a broken JavaScript import, a bad Region, or proof that presigned URLs do not work. S3 evaluated the attempted operation using the signing identity's permissions and did not find the necessary Allow. The policy attached to the user had a name suggesting broad S3 access, but **policy names do not establish the actions they grant**. The actual permission statements matter.

A narrowly scoped identity-based policy for a learning bucket can grant read access to objects in only that bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadLearningObjects",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-private-demo-bucket/*"
    }
  ]
}
```

The bucket name is illustrative. The ARN ending in **`/*`** refers to **objects inside the bucket**. By contrast, the bucket ARN without that object suffix is used for bucket-level permissions such as `s3:ListBucket`. These are different actions and do not automatically imply one another.

Permission evaluation has multiple layers. An identity policy granting `s3:GetObject` is a necessary remedy for the specific missing-allow message in this exercise, but access could still be blocked by an **explicit Deny**, a restrictive bucket policy, other organizational constraints, or applicable encryption-key permissions. A presigned URL does not bypass those controls. The security benefit is that the application can keep the bucket private and delegate **specific, time-limited access**, rather than enabling public read for the entire bucket.

**Important learning distinction:** IAM access and S3 Block Public Access serve different purposes. Allowing an authenticated signing identity to call `GetObject` is **not** the same as making every object publicly readable. Do not disable Block Public Access merely to make a presigned GET work.

### 37. Two errors, two independent causes: today's troubleshooting sequence

The exercise produced two separate failure messages in order. They are useful because each illustrates a different stage of handling an S3 request.

**First: `PermanentRedirect`.** The URL was constructed for **`ap-south-1`**, while the bucket's specified endpoint was in **`eu-north-1`**. S3 indicated that the bucket must be addressed using the proper endpoint. The remedy was to update the S3 client's Region and generate a **new** URL signed for that Region. The next error demonstrated that the incorrect-Region problem had been bypassed.

**Second: `AccessDenied` with “no identity-based policy allows `s3:GetObject`.”** The later URL reached S3 in the correct regional context, but the IAM identity that generated it was not authorized for the requested object read. The remedy was to inspect the actual IAM permission statements and grant **`s3:GetObject`** on the intended objects, without unnecessarily broadening access.

These are **not** interchangeable problems. Widening IAM permissions cannot repair a wrong regional endpoint; changing the Region cannot create a missing IAM Allow. When debugging, read the actual error instead of trying every AWS setting in turn.

The two screenshots verify that the SDK successfully **generated presigned URLs**, and that the browser encountered and then moved past the Region problem to a permission problem. They do **not**, by themselves, verify that the final object download succeeded after the permission change. That end-to-end success is left unconfirmed in the study record.

### 38. Storage security, lifecycle, and costs worth knowing now

Because S3 can hold sensitive user files, a good default is to **keep the bucket private** and grant only the necessary actions to carefully scoped identities. A backend should decide **whether a user is entitled to a file** before generating a presigned link. A short expiration reduces the period of exposure, but does not replace authorization. Publicly exposing an entire bucket to make one test URL work removes this useful boundary.

S3 provides encryption and storage-management features, but they solve different problems. **Encryption at rest** protects stored data under the selected encryption scheme; **IAM and bucket policies** determine who may perform operations; **versioning**, if enabled, can retain multiple object versions and help recover from an unintended overwrite or deletion; and **lifecycle rules** can transition or expire objects under defined conditions. Turning on versioning may retain additional data and therefore increase storage costs.

Choosing a **storage class** also influences cost and access behavior. Standard storage suits frequently accessed objects; other classes can reduce certain storage costs in exchange for retrieval charges, minimum storage duration, or different access characteristics. A temporary practice bucket rarely needs elaborate lifecycle and storage-class engineering, but understanding that object storage has ongoing charges is essential.

S3 costs may include data stored, requests made, outgoing data transfer, and other features used. Keep sample data small, verify pricing and account credits, and delete test objects and unneeded buckets when finished. Deleting a bucket ordinarily requires that its contents be emptied first; if versioning was enabled, object versions and delete markers may also need removal.

The **sensitive access key visible in today's earlier screenshot was exposed**. For secure practice, that specific key should be **deactivated or deleted and replaced**, and the replacement credentials should be stored through a supported profile or temporary-credential workflow rather than inside `index.js`. A key that was published in a screenshot must not be considered safe merely because the account was created for learning.

### 39. How this fits into a real application

Imagine a student using Mathead to access a private exam document. The browser makes an authenticated request to the application's backend: “I want to open this document.” The backend checks the student's entitlement using the application's own user and database records. If access is permitted, it generates a short-lived **presigned S3 GET URL** for that precise object's bucket and key.

The browser then downloads the object **directly from S3**, using the signed URL. The backend need not hold the entire file in memory or stream its bytes through the application server. S3 handles the transfer while the application remains responsible for the access decision and the decision about how long a link should live.

That architecture has limits. A presigned URL can be forwarded during its validity period, so it should not be interpreted as proof that the person opening it is still the original user. For especially sensitive data, the application may need additional constraints or a different delivery design. The lesson is not that every file should always use a presigned URL; it is that **private object storage and temporary delegated access are distinct, composable capabilities**.

The next related topic is **presigned PUT upload URLs**, in which a browser can upload data to a permitted object key without receiving general AWS credentials. The signing method is related, but the HTTP operation and required IAM permission are different. **That lesson belongs to tonight's separate video and is not yet covered as completed material here.**

### 40. Checkpoints to test understanding

1. Why is S3 a better home for durable user uploads than the temporary local filesystem of an application server that may be redeployed?
2. If an object has key `notes/chapter1.txt`, why is asking for key `chapter1.txt` not necessarily the same request?
3. What is the difference between *constructing a presigned URL* and *successfully downloading an S3 object using it*?
4. Why did changing `ap-south-1` to `eu-north-1` resolve the `PermanentRedirect` problem but not the `AccessDenied` problem?
5. What IAM action must authorize the signing identity for a presigned **GET** link, and why does granting it not automatically make the entire bucket public?
6. Why is a presigned URL best treated as a temporary bearer credential rather than as a freely shareable public link?
7. What would you change if a frontend needed to **upload** a file rather than download one? Identify the operation that changes, but leave its complete implementation for the next lesson.

**Sources for this section**

- [AWS — Getting started with Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html)
- [AWS — Download and upload objects with presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)
- [AWS SDK for JavaScript v3 — S3 presigner package](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-s3-request-presigner/)
- [AWS SDK for JavaScript v3 — Complete S3 examples](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_s3_code_examples.html)
- [AWS — Troubleshooting S3 AccessDenied (403)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/troubleshoot-403-errors.html)
- [AWS — S3 pricing and cost factors](https://aws.amazon.com/s3/pricing/)

---

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
