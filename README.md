# monolyth_to_microservices (The page is under development)
In this project we will deploy a monolithic node.js application to a Docker container, then decouple the application into microservices without any downtime. The node.js application hosts a simple message board with threads and messages between users.

<p align="center">
  <img src="https://github.com/otammato/monolith_to_microservices/images/monolith_1-monolith-microservices.70b547e30e30b013051d58a93a6e35e77408a2a8.png" />
</p>

#### Monolithic Architecture
The entire node.js application is run in a container as a single service and each container has the same features as all other containers. If one application feature experiences a spike in demand, the entire architecture must be scaled.

#### Microservices Architecture
Each feature of the node.js application runs as a separate service within its own container. The services can scale and be updated independently of the others.
<br><br><br>
Technologies used for the project:
- AWS Cloud9
- AWS ECS, AWS ECR
- Docker
- Node.JS 
- "Koa" for Node.js 
<br><br><br>

# 1.
In this section, we will build the Docker container image for our monolithic node.js application and push it to Amazon Elastic Container Registry (Amazon ECR)
![alt text](https://github.com/otammato/monolith_to_microservices/images/blob/main/monolith_3-Image-Deployment-to-Amazon-ECR.ef4f8b89baccbd37380998a8d896126df5ed8a3b.png?raw=true)
<br><br>

#### 1.1. Pre-requisites.
##### 1.1.0. I recommend using AWS Cloud9 IDE as it has installed by default all the necessary tools to implement this project
##### 1.1.1. Check if git is installed on your machine, clone the repository with app, unzip files and navigate to the project folder
<pre>
$ git --version
git version 2.32.1
$ git clone https://github.com/otammato/monolyth_to_microservices.git
$ cd monolyth_to_microservices
$ tar -xf lab-files-ms-node-js.tar.gz
$ cd 2-containerized-monolith/
$ ls
db.json  Dockerfile  package.json  server.js
</pre>
<br>

##### 1.1.2. Install "koa" module for Node.js and start a local server using npm start command. "Koa" is a minimalist web framework for Node.js, created by the same team that developed Express. It is designed to be a smaller, more expressive, and more robust foundation for web applications and APIs. Koa aims to be a simpler, more modern, and more flexible alternative to Express, with a strong emphasis on async functions and middleware.
<pre>
$ npm install koa
$ npm install koa-router
$ npm start
</pre>
<br>

##### 1.1.3. Test our app on local server. It returns the object as a response.
<pre>
$ curl localhost:3000/api/users
[{"id":1,"username":"marceline","name":"Marceline Singer","bio":"Cyclist, musician"},{"id":2,"username":"finn","name":"Finn Alberts","bio":"Adventurer and hero, defender of good"},{"id":3,"username":"pb","name":"Paul Barium","bio":"Scientist, cake lover"},{"id":4,"username":"jake","name":"Jake Storm","bio":"Soccer fan, sky diver"}]
</pre>
<br>


##### 1.1.4. Check if AWS CLI is is installed on your machine, upgrade it and add the $HOME/.local/bin directory to your PATH environment variable. By adding $HOME/.local/bin to your PATH, you are making it so that your shell will search for executables in that directory as well.
<pre>
$ pip3 install awscli --upgrade --user
$ export PATH=$HOME/.local/bin:$PATH
</pre>
<br><br>


#### 1.2. Containerizing the monolitic app to Amazon ECS.
##### 1.2.1. Launch an AWS ECR (elastic container registry).
<img src="ECR_create.png" alt="drawing" width="700"/>
<br>

##### 1.2.2. Keep the uri of ECR for later.
<img src="ECR_created2.png" alt="drawing" width="700"/>
<br>

##### 1.2.3. Check the push commands to push a Docker image to ECR registry.
<img src="ECR_push_c2.png" alt="drawing" width="700"/>
<br>

##### 1.2.4. Check the current directory and content of the docker file.
<pre>
$ pwd
/home/ec2-user/environment/monolyth_to_microservices/2-containerized-monolith
$ vi Dockerfile
</pre>
<br>
<br>

##### 1.2.5. Check the commands in a Docker file.<br><br>This Dockerfile creates a new Docker image based on the mhart/alpine-node:7.10.1 base image.<br><br>The WORKDIR command sets the working directory for the rest of the instructions in the Dockerfile. The ADD command copies the files in the current directory (.) to the working directory in the image (/srv). The RUN command runs the npm install command, which installs the dependencies listed in the package.json file.<br><br>The EXPOSE command tells Docker that the container listens on the specified network port at runtime. The CMD command specifies the command that should be run when the container is started from the image. In this case, it runs the node server.js command.
<pre>
FROM mhart/alpine-node:7.10.1

WORKDIR /srv
ADD . .
RUN npm install

EXPOSE 3000
CMD ["node", "server.js"]
</pre>
<br>
<br>



##### 1.2.4. Use this commands for later steps.
<img src="push.png" alt="drawing" width="700"/>
<br>

##### 1.2.5. This command is typically used to authenticate Docker to an Amazon ECR registry so that you can use the docker push and docker pull commands to transfer images to and from the registry. Don't forget to replace [000000000000] with your AWS account ID. Before that, change your directory to 2-containerized-monolith/ 
<pre>
$ cd 2-containerized-monolith/
$ aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [000000000000].dkr.ecr.us-east-1.amazonaws.com
</pre>
<br>

##### 1.2.6. Build your Docker image using the following command. Note the dot in the end of command. It's important.
<pre>
$ aws docker build -t api-repo .
</pre>
<br>

##### 1.2.7. After the build is completed, tag your image so you can push the image to this repository. Don't forget to replace [000000000000] with your AWS account ID.
<pre>
$ docker tag api-repo:latest [000000000000].dkr.ecr.us-east-1.amazonaws.com/api-repo:latest
</pre>
<br>

##### 1.2.8. And finally, run the following command to push this image to your newly created AWS repository. Don't forget to replace [000000000000] with your AWS account ID.
<pre>
$ docker push [000000000000].dkr.ecr.us-east-1.amazonaws.com/api-repo:latest
</pre>
<br>

##### 1.2.4. Now you can see the pushed docker image in your repository. Save the image URI for later to create a task to run it in a cluster. This is equivalent to go to Docker Hub and pull an image.
<img src="pushed_di2.png" alt="drawing" width="700"/>
<br>
<br>
<br>



# 2.
In this section, we will use Amazon Elastic Container Service (Amazon ECS) to instantiate a managed cluster of EC2 compute instances and deploy your image as a container running on the cluster.
![alt text](https://github.com/otammato/monolyth_to_microservices/blob/main/containerized_monolyth.png?raw=true)
<br><br>

##### 2.1. Creating an AWS ECS cluster.
<img src="ECS_cluster.png" alt="drawing" width="700"/>
<img src="ECS_cluster1.png" alt="drawing" width="700"/>
<img src="ECS_cluster2.png" alt="drawing" width="700"/>
<img src="ECS_cluster3.png" alt="drawing" width="700"/>
<img src="ECS_cluster4.png" alt="drawing" width="700"/>
<br>

##### 2.2. Configure a task definition.
<img src="task_def1.png" alt="drawing" width="700"/>
<img src="task_def2.png" alt="drawing" width="700"/>
<img src="task_def3.png" alt="drawing" width="700"/>
<img src="task_def4.png" alt="drawing" width="700"/>
<img src="task_def5.png" alt="drawing" width="700"/>
<br>

##### 2.3. Creating a Load balancer and a target group for LB.
<img src="elb_2.png" alt="drawing" width="700"/>
<img src="elb_3.png" alt="drawing" width="700"/>
<img src="elb_4.png" alt="drawing" width="700"/>
<img src="elb_5.png" alt="drawing" width="700"/>
<img src="elb_6.png" alt="drawing" width="700"/>
<img src="elb_7.png" alt="drawing" width="700"/>
<img src="elb_8.png" alt="drawing" width="700"/>
<img src="elb_9.png" alt="drawing" width="700"/>
<img src="elb_10.png" alt="drawing" width="700"/>
<img src="elb_11.png" alt="drawing" width="700"/>
<img src="elb_12.png" alt="drawing" width="700"/>
<br><br>
return to the previous tab where we were setting up a load balanser, refresh the target groups and choose our newly created target group
<br><br>
<img src="elb_13.png" alt="drawing" width="700"/>
<img src="elb_14.png" alt="drawing" width="700"/>
<img src="elb_15.png" alt="drawing" width="700"/>
<br>

##### 2.4. Configure/update the security group.
<img src="sg_1.png" alt="drawing" width="700"/>
<img src="sg_2.png" alt="drawing" width="700"/>
<img src="sg_3.png" alt="drawing" width="700"/>
<img src="sg_4.png" alt="drawing" width="700"/>
<img src="sg_5.png" alt="drawing" width="700"/>
<br>

##### 2.5. Deploy the monolith as a service into the cluster.
<img src="mon_as_a_service1.png" alt="drawing" width="700"/>
<img src="mon_as_a_service2.png" alt="drawing" width="700"/>
<img src="mon_as_a_service3.png" alt="drawing" width="700"/>
<img src="mon_as_a_service4.png" alt="drawing" width="700"/>
<img src="mon_as_a_service5.png" alt="drawing" width="700"/>
<img src="mon_as_a_service6.png" alt="drawing" width="700"/>
<img src="mon_as_a_service7.png" alt="drawing" width="700"/>
<img src="mon_as_a_service8.png" alt="drawing" width="700"/>
<img src="mon_as_a_service9.png" alt="drawing" width="700"/>
<img src="mon_as_a_service10.png" alt="drawing" width="700"/>
<img src="mon_as_a_service11.png" alt="drawing" width="700"/>
<img src="mon_as_a_service12.png" alt="drawing" width="700"/>
<img src="mon_as_a_service13.png" alt="drawing" width="700"/>
<img src="mon_as_a_service14.png" alt="drawing" width="700"/>
<br>

##### 2.5. Test the monolith.
<img src="mon_deployed1.png" alt="drawing" width="700"/>
<img src="mon_deployed2.png" alt="drawing" width="700"/>
<img src="mon_deployed3.png" alt="drawing" width="700"/>
<img src="mon_deployed4.png" alt="drawing" width="700"/>
<br>

# 3.
In this section, we will break the node.js application into several interconnected services and push each service's image to an Amazon Elastic Container Registry (Amazon ECR) repository.

In the previous two modules, we deployed our application as a monolith using a single service and a single container image repository. To deploy the application as three microservices, you will need to provision three repositories (one for each service) in Amazon ECR.
Our three services are:

- users
- threads
- posts

![alt text](https://github.com/otammato/monolyth_to_microservices/blob/main/monolith_5-Microservices-app-architecture.a0103d1b39c5702fb94cfa20ec7fa29f1bb75e1f.png?raw=true)
<br><br>
a. Client
The client makes traffic requests over port 80.

b. Load Balancer
The ALB routes external traffic to the correct service. The ALB inspects the client request and uses the routing rules to direct the request to an instance and port for the target group matching the rule.

c. Target Groups
Each service has a target group that keeps track of the instances and ports of each container running for that service.

d. Microservices
Amazon ECS deploys each service into a container across an EC2 cluster. Each container only handles a single feature.
<br><br>
### 3.0. Check the content of folder "3-containerized-microservices".
Now our app is split into 3 folders: users, threads, posts, each contains its own Docker file
<br><br>
### 3.1. Creating repositories to push there each docker container.
#### 3.1.1. Create a repository for "users" Docker image.
#### 3.1.1.1. Create a repository for "users" Docker image.
<img src="ecr_depo_users1.png" alt="drawing" width="700"/>
<img src="ecr_depo_users2.png" alt="drawing" width="700"/>
<img src="ecr_depo_users3.png" alt="drawing" width="700"/>
<br><br>

#### 3.1.1.2. Create and push the Docker image from "users" folder to "api-users" repository. Replace [000000000000] with your AWS account ID.
<pre>
$ cd users/
$ aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [000000000000].dkr.ecr.us-east-1.amazonaws.com
$ docker build -t api-users-repo .
$ docker tag api-users-repo:latest [000000000000].dkr.ecr.us-east-1.amazonaws.com/api-users-repo:latest
$ docker push [000000000000].dkr.ecr.us-east-1.amazonaws.com/api-users-repo:latest
</pre>
<br>

#### 3.1.2. Create a repository for "posts" Docker image.
#### 3.1.2.1. Create a repository, create an image and push to a repository similarly to steps from 3.1.1.
<br>

#### 3.1.3. Create a repository for "threads" Docker image.
#### 3.1.3.1. Create a repository, create an image and push to a repository similarly to steps from 3.1.1.
<br>

The result with all repositories and Docker images inside of each:
<img src="ecr_repo_microserv.png" alt="drawing" width="700"/>
<br><br><br>

#### 3.2. Create task definitions.
#### 3.2.1. Create task definition for users
<img src="task_def1.png" alt="drawing" width="700"/>
<img src="task_def2.png" alt="drawing" width="700"/>
<img src="task_def3.png" alt="drawing" width="700"/>
<img src="task_def4.png" alt="drawing" width="700"/>
<img src="task_def5.png" alt="drawing" width="700"/>
<br>

#### 3.2.2. Create task definition for threads
Follow the similar steps as for 3.2.1.
<br>

#### 3.2.3. Create task definition for posts
Follow the similar steps as for 3.2.1.
<br><br>

The result with all task definitions created:<br>
<img src="task_def6.png" alt="drawing" width="700"/>
<br><br>

#### 3.3. Create target groups.
#### 3.3.1. Create a target group for users
<img src="target group1.png" alt="drawing" width="700"/>
<img src="target group2.png" alt="drawing" width="700"/>
<img src="target group3.png" alt="drawing" width="700"/>
<img src="target group4.png" alt="drawing" width="700"/>
<br>

#### 3.3.2. Create a target group for threads
Follow the similar steps as for 3.3.1.
<br><br>

#### 3.3.3. Create a target group for posts
Follow the similar steps as for 3.3.1.
<br><br>

The result with all target groups created:<br>
<img src="target group5.png" alt="drawing" width="700"/>
<br><br>

#### 3.4. Update rules for a load balancer to redirect HTTP requests appropriately.
<img src="lb_rule1.png" alt="drawing" width="700"/>
<img src="lb_rule2.png" alt="drawing" width="700"/>
<img src="lb_rule3.png" alt="drawing" width="700"/>
<img src="lb_rule4.png" alt="drawing" width="700"/>
<img src="lb_rule5.png" alt="drawing" width="700"/>
<img src="lb_rule6.png" alt="drawing" width="700"/>
<img src="lb_rule7.png" alt="drawing" width="700"/>
<img src="lb_rule8.png" alt="drawing" width="700"/>
<br><br>

#### 3.5. Create services.
#### 3.5.1. Create a service for users
<img src="service_ms1.png" alt="drawing" width="700"/>
<img src="service_ms2.png" alt="drawing" width="700"/>
<img src="service_ms3.png" alt="drawing" width="700"/>
<img src="service_ms4.png" alt="drawing" width="700"/>
<img src="service_ms5.png" alt="drawing" width="700"/>
<img src="service_ms6.png" alt="drawing" width="700"/>
<img src="service_ms7.png" alt="drawing" width="700"/>
<br><br>

#### 3.5.2. Create a service for threads
Follow the similar steps as for 3.5.1.
<br><br>

#### 3.5.3. Create a service for posts
Follow the similar steps as for 3.5.1.
<br><br>

The result with all services created:<br>
<img src="service_ms8.png" alt="drawing" width="700"/>
<br><br>

#### 3.5. Disengage the monolith service.
<img src="disengage_m1.png" alt="drawing" width="700"/>
<img src="disengage_m2.png" alt="drawing" width="700"/>
<img src="disengage_m3.png" alt="drawing" width="700"/>
<img src="disengage_m4.png" alt="drawing" width="700"/>
<br><br>


#### 3.6. Test our microservices.
#### 3.6.1. Copy and paste the DNS name of our load balancer in a new browser tab:
<img src="elb_dns.png" alt="drawing" width="700"/>
<br><br>

#### 3.6.2. Then test different services endpoints if they work as expected:
<img src="ms_test11.png" alt="drawing" width="700"/>
<img src="ms_test22.png" alt="drawing" width="700"/>
<img src="ms_test33.png" alt="drawing" width="700"/>
<img src="ms_test44.png" alt="drawing" width="700"/>
<img src="ms_test55.png" alt="drawing" width="700"/>
<br><br>
