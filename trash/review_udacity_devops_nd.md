---
title: "Review: Cloud DevOps Engineer Nanodegree"
image: images/devops1.PNG
showonlyimage: false
weight: 2
date: 2020-05-24T18:00:45-06:00
draft: true
---

On March 15th Udacity launched a promotion aimed to help people affected by the pandemic. (yup! the Covid-19  thing I'm tired of). 


This was about getting a completely free one month access   (30 days) to any of their programs. I took advantage and gave it a try. I enrolled in the DevOps nanodegree and here are my thoughts.


## TL;DR 
The program is worth it if you have basic knowledge of cloud and docker containers. To understand the projects you will need to do some research. 


The course has a lot of emphasis on Jenkins but also touches CircleCi. The things you learn on CircleCI apply to other YAML based platforms (Gitlab CI, Bamboo, Travis, and so on)

I recommend this for advanced beginners that want to do production-level stuff with AWS, Docker, CI/CD, and Kubernetes.

## Nanodegree structure
The program consists of 4 courses and 5 projects:

 * Cloud fundamentals.
 * Deploy infrastructure as code (IAC).
 * Build CI/CD pipelines, monitoring, and logging.
 * Microservices at scale using AWS and Kubernetes

### Cloud Fundamentals
Here you touch the base of AWS: EC2, ECS, S3, Load balancers, SNS, CloudFront distributions. This module also introduces you to the AWS management console. This part is very basic and 80% theory, with some hands-on examples of security groups, IAM roles, and SNS .

If you already have experience with AWS this will be easy for you, but still I recommend you to see what is inside.

>**Project:** for this section consists of a single page static website, deployed using S3.

![s3][1]


### Deploy infrastructure as code
This section is about CloudFormation. Prepare for reading a lot of YAML. This module gives a good understanding of infrastructure as code on the AWS platform. You will create Infrastructure diagrams using LucidChart, which is a nice online tool. The module shows how to translate your ideas to diagrams and use those as a visual guide for your CloudFormation templates. 

The goals for this section, as written on the course page, are:

* _Describe IaaC as one of the best practices used in the DevOps model._
* _Configure basic settings to start using AWS services as an IAM user._
* _Explain the fundamentals of Cloud Formation._
* _Contrast the manual vs. automated provisioning of EC2 instances in a VPC._
* _Use the AWS command-line tool - CLI for necessary activities, such as_ 


This section teaches what they offer. The project was rather interesting and useful. For this section you deploy a Load Balancer with an autoscaling group. This mimics a production setup and you achieve all that via Infrastructure as code

>**Project:** Deploy a High available web app using a Load Balancer and autoscaling group, all this setup is done via CloudFormation.

[comment]: <>

> **Tip:** While debugging your CloudFormation templates use the visual editor AWS offers. Override the default template with yours. The editor is very useful at linting errors on your YAML files and will save you tons of time.

[comment]: <>

>**Tip:** Use YAML for your CloudFormation templates, it is more readable than using JSON.

![Load balancer diagaram][2]


### Build CI/CD pipelines, monitoring, and logging.

This module explains the fundamental CI/CD concepts:

 * continuous Integration
 * continuous Delivery
 * continuous Deployment


You learn how to setup Jenkins on an EC2 instance. Configure different plugins. Store AWS and GitHub secrets on the  Jenkins configuration. Running different pipelines and jobs for different products.

In this section, you learn Ansible for configuration management. You also learn to configure your Jenkins box using Ansible. 

It also introduces a popular tool called Prometheus for monitoring. Finally you use the ELK stack to visualize metrics and alerts.

Ok, don't get me wrong. The rest of the program was useful, but I'm not that comfortable with this module for the following reasons:

* The syntax of Jenkins, based on groovy, is too specific for this tool. The knowledge you gain won't be easy to translate into other platforms as Circle, Travis, Gitlab, and so.

* Jenkins is very popular, but the YAML based SaaS offerings are gaining a lot of popularity due to its usability. (Yes Gitlab CI I'm looking at you RN)

* Jenkins is plugin based and doesn't treat docker as a first-class citizen (you know docker's so hot rn!). Jokes aside, moving to a more flexible Docker-based option instead of native plugins would be nice.

* Many people, myself included, were complaining in the forums about this course lack of clarity as it's very wide, but less depth.

I think Udacity has the opportunity for improving in these aspects and include a more up to date option.  ** But I'm also aware there's a lot of Jenkins at use in the enterprise world, so it might get you an interview easier **

>**Project:** You complete the Jenkins setup on an EC2 instance and build a very simplistic version of a pipeline using BlueOcean and AWS plugins.

![simple pipeline][3]

### Microservices at scale using AWS and Kubernetes

The course starts by introducing you to the serverless world using AWS lambda. You learn to write basic lambda functions in python by using the cloud IDE AWS Cloud9. Then you learn what is docker and how a Dockerfile looks. You end the first part of the module containerizing a Machine learning microservice using Docker.

The third part is about Kubernetes, here you learn about:

 * Containers
 * Pods
 * Clusters
 * Services
 * Nodes

You set up a [local cluster with MiniKube](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/) and interact with it via kubectl, this knowledge would be useful for your capstone project that will run on EKS.
EKS stands for Elastic Kubernetes Service. EKS is the AWS offering for taking care of the Kubernetes control plane for you. You will learn what EKS is, eksctl, and taking care of publishing Kubernetes pods via _Services_

This was the most interesting part of the course, but the one that demands the most research. 

> **Tip:** Kubernetes has a steep learning curve, you'd want to familiarize yourself with the official [introduction guide]( https://kubernetes.io/docs/tutorials/kubernetes-basics/)

[comment]: <>

>**Project:** You package a Flask API into a docker container, after linting both the source code and the Docker file using CircleCi, you deploy it to a Kubernetes cluster.

![k8s][]

### Capstone project

Taken from the official project page:
_In the capstone project, each project is unique to the student. You’ll build a CI/CD pipeline for a micro-services application for different deployment strategies. Students define the scope of the project and select the right deployment strategy based on different business requirements._

Here you will through choosing the right deployment strategy for your project. Most students work with Blue/Green deployments and others with Rolling Release deployments.
This is the widest project since you decide what to deploy, how to deploy, how the pipeline is built, and have to work on the Kubernetes architecture on your own.

The whole course is well structured and very educative, but having to engineer a solution on your own is the *most* educative thing you can do.

![Simple diagram][4]

![Simple pipeline][5]


### Closing thoughts
DevOps is more than automation, is a vast field with a larger tooling ecosystem. It is more about the process, the people, and the journey.

This Nanodegree won't make an expert out of you, but introduces you to the things you need to know, as Cloud platforms, CI/CD, Containers, Orchestrators, and monitoring tools. 

I recommend this program to anyone eager to learn about DevOps.

For feedback don't hesitate on dropping me a line. I'll be more than happy to fix any incomplete or wrong information.

![Certificate][6]

[1]: ../../images/s3-static.png
[2]: ../../images/lb-diagram.jpeg

[ 3 ]: ../../images/simple-pipeline.PNG
[4]: ../../images/capstone-diagram.png
[5]:../../images/capstone-pipeline.png
[6]: ../../images/devops1.PNG

