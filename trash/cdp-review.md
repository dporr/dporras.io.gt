---
title: "Review: Practical DevSecOps CDP"
image: images/devops2.png
showonlyimage: false
weight: 2
date: 2020-08-01T21:20:04-06:00
draft: true
---
## TL;DR

- Won't make you an expert, but gives **very good base**
- Believe them when they say Practical, be ready to actually DO stuff by yourself.
- Short curse, concise, very informative
- Teaches how to transform a regular CI/CD pipeline into a production ready DevSecOps* pipeline
- Good for beginners in DevOps, but at least you have some IT background
- They mention commercial products, but the course is focused on open source tools.

## Context

During early 2020 (the good old days :cowboy: ) I moved from my role as pentester to be an Application security dude at company XYZ.  I started to get involved with a new workflow and saw too many ways the job could be automated. Working in an Agile cloud-based environment and with the awesome engineers the company have writing code at lightning speed, the AppSec team needed to be at the same pace. I started my research on how, why, and the current trends in similar cloud-based businesses are doing automation and large scale security... The magic word popped up: *DevSecOps*

Doing searches on LinkedIn I found practical-DevSecOps. After following them they published a giveaway where I participated and was one of the lucky winners, out of 3, for a scholarship for their CDP (Certified DevSecOps Professional). The scholarship included 30 days to their labs + course content + 1 certification attempt.

## Course content 

The course consists of 9 modules including:

* Introduction to DevSecOps (This is the core for the theoretical part)
* Secure SDLC and CI/CD pipelines  (This is the core of the technical part)
* SAST (Static application security testing) in CI/CD pipeline
* DAST (Dynamic application security testing) in CI/CD pipeline
* Infrastructure as code (IAC) and its security
* Compliance as code

Other modules were included, too. You can see the full syllabus here:
https://www.practical-devsecops.com/certified-devsecops-professional/

## Course approach

After a short video, about 10 mins long, you always have a practical challenge to test the technical concepts, also there are examples of the things you need to accomplish. The course includes a PDF guide for the labs with code snippets you can directly copy and paste to try yourself, but I suggest you type on your own and read what the code is doing this will be useful for the exam and improves your overall understanding.

So get ready to automate security controls inside CI/CD pipelines, run continuous security scanning, identify weaknesses in your own source code and third-party libraries, understand when a tool is right to be embedded into pipelines, or is more suited to an offline/parallel run. Capture the artifacts/scan results, centralize them via vulnerability management software, check compliance, and automate system hardening... Sounds like myriad things, but the process is straight forward and fun.

## Staff support

One of the things I found outstanding is the support you get from Practical-Devsecops staff. Although the lab instructions are very clear, there might be difficulties, questions related to the labs, or questions for enhancing your understanding of a specific topic. For those scenarios whenever you ask on the private slack channel there was someone assisting. For me, the support worked virtually 24/7, and this is especially good for the sort of informative discussions that happen on the slack.

## The exam

The exam will challenge you to build a complete CI/CD pipeline including all the topics you already practice in the lab and course. You start with a bare  CI pipeline, defined in YAML, and sequentially improve it through 5 practical, well-demarcated, challenges.

The lab will give you 85% of the foundation on what is needed to complete the exam, but with some extra requirements that need to complete on your own (Not that complicated if you understood the core course concepts).

Short story: There was a specific challenge that I solved by replicating a short python script and modifying it to the specific problem I was solving. I was aware of this script since I observed carefully each tool, container, and instructions in the course. The lesson here is: If you observe what they are trying to teach you will be good for the exam.

## Final advice on passing the exam

There are various references and extra materials in the course, so take your time to go through each link, tool, and concept. You don't need to be an expert, but at least understand the tool and get comfortable with it.

For example, I didn't know a thing about Gitlab CI, Ansible, Inspec, and other tools. They did a good job introducing the tools, but I reviewed other blog posts and video courses to get a good grasp. In the end, a better understanding of the tooling and concepts I got was the key to winning the Gold coin (After passing the exam with a good grade).

So don't be afraid, this course is well suited for people that are willing to read and try the concepts in a practical fashion. At least be comfortable using the terminal and have a general understanding of scripting (read and replicate basic python code is enough), CI/CD process (Conceptually), and basic owasp top 10.****