[[Cloud Computing]]
****
**Table of Contents**
```table-of-contents
```

****
## Problem

Actually, **It's hard to set up containers**
- We need an environment to build beforehand
- We need a management layer
- We need back-end


****
## Serverless

With serverless, we only **provide code for the business core features**
All management and execution is provided by the cloud platform
	*From execution environment to service availability*

`Function-as-a-Service + Backend-as-a-Service = Serverless`


### Backend-as-a-Service (BaaS)

We leverage backend services without building nor maintaining them.
	*Database servers, message queues, (object) storage... Most application architectures are similar as they require those common backend components*

This way, developers can focus on the business core.
	*Those services are (supposed to be) better served by the cloud provider, offering quick scale and mutualisation...*


### Function-as-a-Service (FaaS)

The purpose is to **run backend code without long-lived servers**.
	*We spawn execution environment on-demand, and all of this is managed by our Cloud Platform.*

The **Unit of Execution** is a function (block of code).
	*We now follow the logic that our application will be **event-driven***

==Note that FaaS and Micro-services are different:==
![[faas.png]]

FaaS has several benefits:
- **Elasticity**: granularity of the request handler
	*Can even do "up and down to zero" if not needed at a specific time*
- **Deployment**: Just write code and upload
	*Quick experimentation, quick update as well*
- **Cost**: Pay only the compute time you really need
	*No request = no function running = no resource = no cost
	So,  it's pretty much `Cost = Compute Time x Reserved Memory`*


****
## FaaS Application Architecture

It follows the straightforward **Extract-Transform-Load (ETL) paradigm**: Get data, process data, output data
![[etl.png]]
	*As mentioned above, it is event-driven: Execution is launched only when a request arrives or a trigger is fired*

An important characteristic of the functions we provide is that they are **STATELESS**.
	*If we want to keep track of a request's state, we have to rely on an API Gateway*

We obtain the following ETL workflow for our serverless architecture:
![[ESIPE/Semestre 5 (Pavie)/Cloud Computing/03 - Serverless Computing/architecture.png]]

We have provided environments so our code can run
	*NodeJS, Java, Python...*


****
## OpenWhisk

**OpenWhisk** is an open-source, serverless platform that executes functions in response to events, enabling event-driven architecture for cloud applications.

ETL looks like this with OpenWhisk:
![[openwhisk etl.png]]

And here is an overview of the internals:
![[openwhisk.png]]

This is how a function is invocated:
0. Authentication, other various tasks etc... 
1. A new docker container is spawned with runtime
2. We inject our action code into it
3. Code is executed (with parameters)
4. Result is retrieved once it's done
5. The docker container is destroyed


### Function container management

Steps above works, but step 1 and 5 can be optimised.
Spinning up a new container for a single request is pretty slow (around 400ms). Instead, we can try to re-use our environments !

A **Cold Start** is when a serverless function starts **from scratch**
	*initialising the runtime environment and loading dependencies. Slow...
	We do this when no container is available*
A **Warm Start** is when a previously used runtime environment is re-used
	*40x faster! If runtime containers are available, we use this*

In order to set up this cold-warm logic depending on our running containers, we need to set up a **container pool** (similar to a Thread Pool in an ExecutorService):
- We have a pool of pre-warmed containers
- Containers kept warm use resources but are not billed to the user!


****
## Limits of Serverless

One issue we mentioned above is the possible latency of cold starts.
	*This can be partially fixed via our container pool and the use of warm starts*

Another major issue of serverless is its compatibility with serverful applications
- What if we want to make a stateful application ?
- What if we want to build a massively parallel application ?
	*Since functions are isolated from each other, MPI and concurrency seems impossible*
- FaaS is not fit for long-running computations
	*Will end up costing a lot while being less efficient...*

A last "issue" is that serverless is still new! There is a lot of ground for experimentation (but for now, a lot of problems still persists...)

**So, Serverless is clearly not suited for all applications (yet...)**
