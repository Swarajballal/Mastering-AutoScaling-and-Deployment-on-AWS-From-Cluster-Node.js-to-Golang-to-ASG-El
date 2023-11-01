# Mastering-AutoScaling-and-Deployment-on-AWS-From-Cluster-Node.js-to-Golang-to-Auto-scaling-groups-Elastic-BeanStalk

### AutoScaling Backends

### Making Node Multi core
### Deploying to aws ec2 instances
### Golang vs Node.js
### Understanding images, launch templates and ASG.
### Elastic Beanstalk

0. Bare minimum way to deploy any production worthy app. </br>
1. Understanding autoscalling, why do you need it. </br>
2. Understanding load balancers (popular system design question) </br>
3. AWS way of doing LB/autoscaling. </br>
4. Understanding Backpacks deployment process. </br>

### Single threaded Vs multi threaded languages

Node.js => single threaded </br>
Golang/Java => multi threaded

Node.js or Python are not thar powerful you cant use all the processing power of a machine. </br>

Golang/Java/Rust are multi threaded languages. </br>

Golang is better then node js first in terms of speed a for loop in golang is faster then a for loop in node js.
yes Bun has made node js  is faster but still bun isn't stable yet as of today october 2023. Golang also does multi threading by creating sub routines. Java/Rust lets you create new threads. we can distribute load in multi threaded language. </br>


#### Node js syncronous

![synchronous-nodejs-code](image.png)

it starts a request and then it waits for the response.then it starts another request and then it waits for the response. so and so forth its not starting multiple process parallely  </br>

Difference between syncronous and asyncronous is that let say a asyncronous database call is made then request from backend will go to database to fetch data by the time the data comes back to backend request node js will start another request till the time the data comes back from database. and will be done for subsequent request such that a single core or thread can handle multiple request without keeping thread ideal unlike synchronous nature where it waits for the request to come back and then process the next request. </br> </br>

![asynchronous code](image-1.png)


Even though this is async code still we observe that one request is made it waits for the response to come back and doesn't start another request it is not behaving like async nature
because of how Browser sends request to backend. Chrome for example batches request and sends it to backend. so force request to send right we can use ab to send 1000 request from the terminal </br>

ab -n 1000 -c 16 http://localhost:3000/ </br>

![async code with sb](<Screenshot 2023-10-31 155452.png>)

here as you can see we passed many request at a time see below the request went and didnt wait for the response instead kept sending request and when later on recieved the response </br>

![async response](image-2.png)

In golang subroutines are light weight version of threads much more easy to do context switching, context switch between subroutines is much more faster then context switch between threads. So in Golang you have to explicitly mention that start this in different sub routines only then golang starts it into new thread</br> 

So whenever a request comes it starts in a different go subroutine, so starting in new subroutine the maintainer of the http wrote that logic.

Syntax to start a new sub routine in golang is </br>

```go

go funcName()

go func() {
    // Your code here
}()

```
<br/>

![Go multi thread working](image-3.png)

In go we can see that all the request are being processed parallely in multi cores. </br>

using ab we see all the cores are being used at 100%. it distributes load across all cores </br>

![Alt text](image-4.png)

### Can we do same in node js ? can we spawn multiple threads in node js handle multiple request/parallel execution ?

This can be done using cluster module in node js. </br>

How does cluster module works ? </br>

Let say we execute a node js file ther very first time we execute it. Cluster.Master will be true because we started executing the file. but we also get no of cpu's count or cores in the first run const cpuCount = os.cpus().length; so the rest of the cores/cpu's in cpu's count will be started as cluster.fork() basically new processes will be started and for them cluster.isMaster will be false. So parent cluster will just fork child clusters and child clusters will be the one executing the express servers. If child dies you can restart the processes or kill whole processes do health check etc. so here after the first time if condition won't be executed only else will be executed by child processes.</br>


```javascript

import cluster from "cluster";
import os from "os";
import { app } from "./express-app";

const port = 3000;

const cpuCount = os.cpus().length;

if (cluster.isMaster) {
  // Create a worker for each CPU
  for (let i = 0; i < cpuCount; i++) {
    cluster.fork();
  }
} else {
  app.listen(port, () => {
    console.log("listening on port 3000");
  });
}

```

Since there are 8 cores in my machine so 8 child processes are started. </br>

![8 core started in cluster](image-5.png)


After passing requests using ab we see that all the cores are being used at 100% by a node js process </br>

![node js process with 100% cores used](image-6.png)

### What is Autoscaling ?

Creating a AWS Instance as a construct which we can start and stop to do autoscaling mostly it is done by kubernetes or container orchestration 

### Interview Question
How is Kubernetes different from Vanilla autoscaling ? </br>
 Kubernetes(alreay have a cluster of pods which you turn on off) there is more like a incremental upgradation of pods.so you dont notice a sudden spike its when 20%-50% of the traffic is increased then you increase the pods by 20% or 50% and then you decrease the pods by 20% or 50% when the traffic is decreased. so it is more like a gradual increase and decrease of pods.Whereas the vanila autoscalling (increasing and decreasing EC2 instances on demand) is more like a sudden spike in traffic and then you increase the number of EC2 instances and then when the traffic is decreased you decrease the EC2 instances.So if you have multiple processes its better to use kubernetes and if you have single process then you can use vanilla autoscaling.so a kubernetes cluster have many pods so one pod can run db one can run backend and so on and so forth and communicate using ingress. 

 ### Vanilla Autoscaling using ASG (Auto Scaling Group) in AWS

Hotstar also uses ASG.

##### Deploying to aws 

1. EC2 Machines
2. images
3. Launch templates
4. target groups
5. ASG
6. Elastic 
 
Lets say you create an EC2 instance and you instll node , nvm, npm and clone your code build it to create a dist folder and you create image using image and templates --> create image/clone of machine allocate space to it 8GB default so you can run start or pm2 start on that image/machine so to do this we use launch templates (template to launch more instances). Give name to image in Application and OS images you surf through aws marketplace to find the image you want to use like nginx etc. since we created our own image we will go to
My AMI's and seleect owned by me and select the image we created. its becomes tough to say when the instance starts also run the pm2 command and here docker is easy to use. AWS does provides a way to run a command when the instance starts so select a machine then
a key-pair to ssh into individual machines select a security group you created if you want your final instances to keep port 3000 open , they don't have to but if you want to do it. Usually there is a loadbalancer in between, client hits the loadbalancer which exposes the port 3000 so that cleint dont hit backend servers directly they go through loadbalancer as it should be client should not directly go to backendserver so loadbalancer should have access to backend servers, load balancer ideally should be internet facing and backends should not be internet facing only to debug at first you can give internet access using security groups to find the right one go to you instance then security groups then in launch vizard you will see the security group you created. after this go to advance detail section so user data is the place where you write the command you want to run as soon as images are created or instances are created. now this user-data dont have access to node , npm, pm2 etc so you can't directly put the command to run the server. you have to explicitly mention the path to node, npm, pm2 etc. because they don't know what pm2 , node resolve to they dont have that path. so you have to write binary something like :

```bash
#!/bin/bash
export PATH=$PATH:/home/ubuntu/.nvm/versions/node/v20.9.0/bin
PM2=/home/ubuntu/.nvm/versions/node/v20.9.0/lib/node_modules/pm2/bin/pm2   // put PM2 in path
export PM2_HOME="/home/ubuntu/.pm2"
su-ubuntu -c "cd /home/ubuntu/week-20/part-4-multi-core:             // super user as ubuntu and then run cd command
nodejs/;PATH=$PATH; PM2_HOME=$PM2_HOME $PM2 start npm -- start-u ubuntu"    // ';'for telling the next command and also put the whole Path(from above) in PATH variable(in this line Path)  here PM2_HOME(this is the environment variable pm2 uses incase it is not abel to find its config)=$PM2_HOME(where the config is present as see above PM2_HOME which has the path for pm2) here $PM2(points to exact path of pm2 in  second line above) start npm -- start-u ubuntu(command to start it)
// 'or'
/home/ubuntu/.nvm/versions/node/v20.9.0/bin index.js     //complete path of binary and then index.js
```
Why do we need launch template? when we have a image already because maybe we want each image with different security groups or different commands to run so we need a template to launch instances where we can specify the image, security groups, user-data etc according to our needs. image is just a snapshot while template is similar to image and more information then a image</br>
if you start instances delete the first instance or it will be a additional instance running so if you start 10 instance from a template so in total there will be 11. </br>
we can also write these things in deploy.sh file and then run the file in user-data. </br>
after ssh into ec2 instance run the command which node to see the path to node binary.  </br>
So the above method is good but docker makes it easy to do this. this is for raw node js/go lang/ java </br>
Now our launch template is ready. to launch it you can go to launch template dashboard click on instance abd then actions and laun ch instance from template also specify the number of instances you want to start</br>
Now we need a construct who can start and stop instances on demand.</br>

### ASG (Auto Scaling Group)

ASG takes template as an input and gives you simple constructs. We can take the launch template and directly go to an auto scaling group but since our app has a loadbalancer it is a internet facing application if there was application where users dont hit the loadbalancer directly then we can go to ASG directly. For example a transcoder for video doesn't have to go to loadbalancer since it is not internet facing. Also if we directly create a ASG during the process of setup it will ask if your app is internet facing or not but since out app is on port 3000 and not on port 80 it might run into issues if we dont create target group first. 
lets try creating a ASG directly and this should be the last step if we are using a non internet facing app. Give auto scaling group name select the launch template or base image. as you app grows you might add different technology so then you will go back to your launch template and modify the current template and change to new image you might add git pull command in user-data but if image completely changes then you will create new instance create new image modify the launch template and in modify group select the new version of launch template. first time default version is 1. in availability zones and subnet select the sub nets across which you want to distribute the load you can select multiple regions. next will be configure advance options that will have load balancing since we need one we will select Attach a new load balancer. </br>
AWS has two types of Load Balancers one is at network layer and one is at application layer. Network layer to handle TCP,UDP.TLS etc and application layer to handle HTTP,HTTPS etc. since our app has http request we need application layer load balancer. Load balancer could be internal or internet facing. internal load balancer is used when you have an application that should be used by another AWS application of yours. Internet facing load balancer is used when you have a application that should be used by the internet. Protocol is HTTP and port is 80. since our application is on port 3000 but we can't do this because if we do that load balancer will start listening on port 3000 but Users will hit the port 80 of URL and not port 3000 hence we should have first created a target group. then default routing (forward to) will complain and here you have to select the target group or create a new one but it won't have the right port 
Last thing is Configure group size and scaling policies here you give a desired capacity, minimum capacity and maximum capacity or you can select Scaling policies which lets AWS decide or control the desired capacity, minimum capacity and maximum capacity according to the average CPU utilization here we can set average CPU utilization target value like 50. so if the average CPU utilization is 50% then it will increase the number of instances and if the average CPU utilization is 50% then it will decrease the number of instances. after few next clicks to see if you want to add tags like if instance go up if you want to listen to it etc you check instance management in auto scaling group if a new instance is started and main ec2 dashboard to see other ec2 instance with the new one we can delete the old one not needed.  
</br>

![Architecture](image-7.png)


So Steps that we did to deploy our app to AWS </br>
1. EC2 Instance </br>
2. Image </br>
3. Launch Template </br>
4. Auto Scaling Group with target group and internet facing load balancer </br>

This is enough to deploy a app to AWS. This is good method for non internet facing applications through which you can control the workers like Leetcode where if there are many request you want to increase the number of workers and if there are less request you want to decrease the number of workers. so this good for offline since desired capacity, minimum capacity, maximum capacity can be controlled programitcaly is a good use case</br>

Problems: Since we gave the Port 3000 during creation of Load Balancer let it automatically select it the http port that will be exposed to user was set to be 3000 but we want it to be 80. so here target group acts like a listner so whenever a request comes to port 300 forward it to target group which has multiple EC2 instances. hence we should have created target group first on our own and not just go with ASG process directly. So any request that comes to load balancer forward it to registered target group you can see this in target group dashboard. Target groups also checks health of the instances and only forward request to healthy instances. </br>
How does it check health of instances ? </br>
There is a health check section, where it send a http request at the path at port 80 with some timeout of 5 seconds and if it gets a 200 response it will consider it healthy. else it will consider it unhealthy. </br>
We will also get unhealthy using above since the port it is hitting is 80 and not 3000 and we have set the port to 3000 in our app so change to port 3000 in health check. 

### How to minimize this Complex process ?
Elastic Beanstalk is a service provided by AWS which does all this for you. </br>
Elastic Beanstalk under the hood uses ASG, ASG under the hood uses launch template, launch template under the hood uses image and image under the hood uses EC2 instance. </br>

To debug EWS we use logs on Logs stash, loki, data dog etc to aggregate logs for multiple instances. </br>
There is not point of doing clustering in next js since it does edge caching which puts app in multiple locations. </br>

Try to scale horizontally as much as possible. that means more number of smaller machines. instead of less number of bigger machines. for cost efficency also </br>
