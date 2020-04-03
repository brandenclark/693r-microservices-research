# Microservices

## Microservices as a whole

### What are they

Microservices are an architecture style to build a larger application from a set of smaller services/applications. There are many benefits to microservices that can help improve an application's architecture. With the greatest focus being "enabl[ing] the rapid, frequent, and reliable delivery of large, complex applications...and enabl[ing] an organization to evolve its technology stack" (_<https://microservices.io/index.html>_) This happens when the supporting microservices are:

- Highly maintainable and testable
- Loosely coupled
- Independently deployable
- Organized around business capabilities
- Owned by a small team

Microservices are usually characterized by their decomposition of traditionally monolithic applications across business subdomains or shared capabilities (i.e. services based on how the organization is structured vs what problems the organizations solves). However, really any aspect of an application architecture can be pulled out into it's own service.

### Making the decision to use them

It all comes down to the balance between benefits and overhead for managing an increased number of services. Microservices take more effort to manage/automate and the increased moving parts introduces new concerns with networking, availability, and security. However, that extra cost can increase development speed, reduce application complexity, increase scalability for individual components, and increase the correlation between organization purpose and technology structures.

### Resources

- <https://microservices.io/index.html>

## Containers

Containers are ways to package software applications into a standardized unit that can be run in isolation on an operating system without the need for a virtual machine. This makes the software applications simpler to share and deploy, while reducing the overhead cost for isolating found with VMs.

!["Containers vs VMs"](images/containers-v-vms.png "Containers vs VMs")

XKCD also can show us the difference between using VMs (two phones) vs Containers (one app).

!["xkcd - Containers"](https://imgs.xkcd.com/comics/containers.png "xkcd - Containers")

The packaging of applications in containers creates an abstraction of the environment the application is run on so the application can be predictably ran in the cloud, data centers, or a developer's personal system.

The portability of containers also increases the ability to share container images with an organization or community. Docker Hub is one registry where public and private images are shared, but an organization can also host their own registry of images.

This portability has also led to the ability to build images on top of existing images, and combine images into a compacted app that runs all the images inter-connectedly.

The most common container is Docker. It is supported across all major platforms: Unix and Windows systems, and used prolifically across AWS, Google Cloud Platform, and Microsoft Azure.

All these fact help show that containerizing our microservices has a lot of benefits for deployment and development.

### Resources

- <https://docker-curriculum.com/#dockerfile>

## Orchestrators

The magic portability and predictability from containerization has also opened the door for the orchestration of containers. (e.g. we're able to automate how we manage, scale, and maintain our running images across all sorts of environments). This is because containers effectively guarantee that the application will run the same no matter where it is ran at. The most common orchestrators are _Kubernetes_ and _Docker Swarm_. Both of which come when installing Docker Desktop.

Wherever images are run, orchestration makes it easier to make sure images stay available and on task. So when we make many small services we want to run together we have a way to keep everyone alive and happy.

### Terminology

Here are a few useful terms (as used with Docker Swarm) for understanding orchestration components:

- Swarm
  - a collection of Docker hosts which work together to meet its configured optimal state (number of replicas, network and storage resources, and external ports)
- Node
  - an instance of the Docker engine participating in the swarm
- Manager
  - a node specifically designated to manage swarm membership and delegation
  - swarms start by giving a service configuration to a manager node, who then dispatches work to worker nodes
  - when multiple manager nodes are configured, they elect one leader to control orchestration duties.
  - can also be a worker
- Worker
  - a node running services or tasks in the swarm
- Service
  - the definition of tasks to execute on the manager or worker nodes
  - it includes the images and commmands to be execute inside running containers.
  - services can be replicated across multiple containers to meet redundancy, reliability, and availability needs
  - for global services, the swarm runs one task for the service on every available node in the cluster
- Tasks
  - a task has a container and related commands to execute
  - it is the basic scheduling unit of swarm
  - managers assign tasks to workers according to the service definition
  - once a task is assigned to a node, it cannot move to another node and can only run on the assigned node or fail
- Load Balancing
  - request to public ports are routed to the proper services by the swarm manager
  - the manager uses internal load balancing to distribute requests among configured workers

### Resources

- <https://docs.docker.com/get-started/>
- <https://docs.docker.com/engine/swarm/key-concepts/>

## API Gateway

One issue created by microservices is the change for clients from hitting just one monolithic endpoint to now various endpoints for different microservices. Just look how messy the data flow for this  mobile client is!!

!["w/o an API Gateway"](images/no-api-gateway.png "w/o an API Gateway")

An API gateway helps solve this problem by creating a single point of contact for clients that will handle request routing, response composition, and protocol translation. This helps remove complexity in the client and different services as the API Gateway can handle the details of different client types and requirements.

!["w/ an API Gateway"](images/yes-api-gateway.png "w/ an API Gateway")

### Implementing an API Gateway

- Performance and Scalability
  - API Gateways do best when they are built to support asynchronous, non-blocking requests.
  - Popular technologies supporting this on the JVM side are Netty and Reactor, while Node.js has many options that can fit this flow.
- Using a Reactive Programming Model
  - The previous technologies succeed because of their ability to perform reactive operations. This means that they are able to handle independent requests concurrently, and manage dependent requests without entering callback hell thanks to reactive abstractions such as Futures and Promises.
- Service Invocation
  - Microservices are inherently distributed and need to bridge this gap through either an asynchronous message-system (with a message broker* or directly between services), or a synchronous mechanism (HTTP).
  - It's very common for both to be used within the same microservices environment, including having multiple flavors of each process type.
- Service Discovery*
  - The distributed network means that ip addresses and ports of all the services needs to be tracked. There's not a manual solution since we are harvesting all the benefits of scalable services that grow, move, and shrink as we need them to.
  - This can be done with service or client side discovery with technologies such as Apache Zookeeper.
- Handling Partial Failures
  - When things go south for one dependency, gateways need to be able to fail gracefully, and there's many ways to implement that process.
  - The most popular way is the circuit-breaking* pattern, which stops the client from waiting needlessly for an unresponsive service. If the error rate for a service exceeds a set threshold, the circuit breaker is tripped and all requests using that service fail immediately for a specified period of time. With the circuit breaker checking to see when it can resume operations based on its configurations.

*these topics have their own sections later on

In short, an API Gateway is a great asset for microservice environments because of the single point of entry to access services, the customization it can abstract away from clients and services, and protecting service health through handling failures and caching data.

### Resources

- <https://www.nginx.com/blog/building-microservices-using-an-api-gateway/>

## Service Discovery and Monitoring

All real-world applications have some sort of communication outside of themselves. With microservices, this inherent connection attribute is amplified by the sheer increase in the number services and further compounded by the complexity of the scaling of instances for each service.

Service discovery and monitoring is the mechanism microservices use to manage these connections and keep the whole environment operating smoothly at any scale. Historically, service discovery has its roots in the HOSTS.TXT file, which was replaced by DNS as the internet grew exponentially. Today, service lifecycle can be largely be measured in minutes and seconds as services are spun up and shut down. This led to the need for a dynamic resource to track service addresses and ports within the contained microservice environment.

Today, there are three common approaches to service discovery:

1. Using existing DNS setup
2. Using a generic key-value datastore (e.g. Apache Zookeeper, Consul, etcd)
3. Using a specialized service discovery program (e.g. Netflix Eureka)

The most popular are options #2 and #3. #2 seems to be more prevalent as companies had existing tools that could handle service discovery, and they implemented those when their service discovery needs came up instead of finding/building a new service like #3. I spent more time trying to figure out the specifics of Eureka than I liked too, and still can't tell what it does special outside of being primarily a better solution if you are focused on availability over consistency and your microservices live in AWS.

### Process

The Service Discovery lifecycle is broken down into two main components: Service Registration and Service Resolution. Registration is where microservices let the discovery mechanism know their location and health status. Resolution is where clients access the mechanism to find an available, healthy service to fulfill their needs.

### Additional features: Client-side discovery

Well everything I've summarized so far involves clients accessing services through an external mechanism, there are add-ons to this methodology to make clients directly aware of the registry. This enables them to do their own load-balancing without routing the request through a middle-man. Really, this option just boils down to your situations balance between network or application complexity.

### Resources

- <https://www.datawire.io/guide/traffic/service-discovery-microservices/>
- <https://microservices.io/patterns/client-side-discovery.html>

## Load Balancing and Shedding

### Resources

## Circuit Breaking

### Resources

## Event Messaging

### Resources
