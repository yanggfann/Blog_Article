# What’s the difference between EC2-on-ECS and Fargate-on-ECS?

## Terminology

**ECS** - Elastic Container Service

**EC2** - Amazon Elastic Compute Cloud

**Fargate** - Serverless compute for containers

**Task** – A task is the lowest level building block of ECS. I like to think of these as runtime instances of your docker images.

**Task Definition** – A task definition is a template for Task. It is the specification that defines **the memory, cpu, and networking** requirements of your task executable. In the task definition, you also specify the Docker image that you’d like to run. Typically, most folks using ECS rely on ECR or Elastic Container Registry to source their images.

**Cluster** – **A cluster is just a logical grouping of Tasks or services**. Clusters only really make sense in the context of EC2 machines (since you run your cluster on 1 or more machines), however they are also required when using fargate.

**Service** – **A service is managed component of ECS that allows you to run a number of Tasks over long periods of time.** You define the Task Definition that you’d like to use to form the service, and the number of Tasks you’d like running at any time. You can also set up autoscaling with Services to respond to fluctuating workloads to support even higher scale.

To understand the difference, let’s divide the ECS service into two responsibilities:

- Managing the lifecycle and placement of tasks
- Running containers

## ECS

ECS is a container management service. First, ECS is responsible for managing the lifecycle and placement of tasks. A task is usually made of one or two containers that work together, e.g., an nginx container with a php-fpm container. You can ask ECS to start or stop a task, and it stores your intent. However, ECS does not run or execute your container. ECS only provides the control plane to manage tasks.

## Running containers
To run containers, you have two options. You can use **EC2 instances**, or you can use **Fargate**. Both options work together with ECS. 

The primary difference between these two options – **self-managed infrastructure (EC2)** vs **AWS managed infrastructure (Fargate)**.

**Scaling** container instances is a **challenge** for **EC2**. That’s why we recommend using Fargate.

### Diagram for EC2-on-ECS vs Fargate-on-ECS

![EC2-on-ECS vs Fargate-on-ECS](./pictures/EC2-on-ECS%20vs%20Fargate-on-ECS.png)

Clusters are just pools of resources. Keep in mind that when you create a cluster, you must specify **either** EC2 or Fargate as your launch(run) type – you cannot use both.

#### EC2-on-ECS

When using EC2, you specify the EC2 infrastructure that you’d like to use to **run** your tasks. You can think of **each Container Instance as a EC2 machine**. EC2 machines have fixed resources in terms of cpu and memory depending on the type you provision.

**Depending on the specs of your EC2 machine, you can run 1 or more ECS Tasks within it.**

**For example** – if I had a EC2 machine with 2 vCPUs and 2048MB of memory, I can potentially run at most 2 tasks that required 1 vCPU and 1024MB each.

**You basically just slice your EC2 up into resource partitions.** Using this approach, you can have many tasks running concurrently on the same EC2 machine / container.

#### Fargate-on-ECS

Fargate on the othe rhand performs a lot differently. **In Fargate, there is no notion of hardware** – users just define the number of Tasks they would like to deploy based on a Task Definition. Behind the scenes, AWS maintains a set of machines capable of running your Tasks, and leverages them to do so.

In the Fargate model, you don’t need to care about **where** your Tasks run, you only need to care about **how many** and **when** you want them to run.

## When to use what

### EC2-on-ECS

- You have existing EC2 hardware that you want to use
- Sustained and predictable tasks with high utilization (services)
- Great for control

### Fargate-on-ECS

- Spend less time on setup and maintenance
- Adhoc jobs with variable utilization
- Great for flexibility

## Service load balance

Elastic Load Balancing supports the following types of load balancers: Application Load Balancers, Network Load Balancers, and Classic Load Balancers.

Amazon ECS services hosted on Amazon EC2 instances support the Application Load Balancer, Network Load Balancer, and Classic Load Balancer. 
Amazon ECS services hosted on AWS Fargate support Application Load Balancer and Network Load Balancer only.

### For AWS

Classic Load Balancer support IdleTimeout: https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/config-idle-timeout.html
Application Load Balancers support IdleTimeout: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html
