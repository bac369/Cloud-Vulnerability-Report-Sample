# Homework 3: Deployment Gone Wrong

## Introduction

You work for a company which develops a credit card processing application.
Now that the web application is fixed and ready, your company wants it
deployed in a scalable, reliable, and secure manner. To do this, your
company hired Shoddycorp's Cut-Rate Contracting to containerize your
application, then deploy it in a way that ensures availability and security.
What they delivered, falls quite short of the mark.

What Shoddycorp's Cut-Rate Contracting provided was a deployment
that *almost* works. They containerized the application, the database, and an
Nginx reverse proxy all in Docker. They then created Kubernetes yaml files to
run these containers in a Kubernetes cluster, and configured them to talk to
each other as needed. 

However, upon further inspection we can see that they didn't quite do things
right. Your company must comply with various cybersecurity standards and frameworks
and must attest that the application is secure against a set of security benchmarks.
It seems that the contractor may have failed to meet all the regulatory requirements. 
All-in-all, it's a mess.

## 2) What to Submit
---
Submit your AppSec2 repository along with detailed lab report, with screenshots, to describe the specific actions taken to complete each task. Enough detail should be provide that enable the recipient of your report to blindly reproduce your actions to achieve the same outcomes. Provide explanation to the observations that are interesting or surprising.

All lab tasks should be performed in the provided NYU-AppSec/CSGY9163 Ubuntu 20.04.3LTS virtual machine. Each task is required to include a minimum of 1 screenshot to prove your individual completion of the task. **Every screenshot is required to include the date and time displayed in your virtual machine; otherwise, credit for the particular task will be removed.**

For tasks involving source code or exploit code, include the important code snippets followed by explanation. Simply executing code without explanation will not be eligible for credit.

**Your report _must_ be written in markdown.** Create a folder in your AppSec2 repository, using "Report" as the new folder's name. Within `Report/`, create a sub-folder called "Artifacts" within it. Store all of your screenshots and related laboratory artifacts within the "Artifacts" sub-folder, and include **one** markdown file in the "Report" sub-folder, which will contain your documentation for both, Part 1 and Part 2, of this lab.

Your repository should now include the following file structure:

    - GiftCardSite/
        - ...
    - Report/
        - Artifacts/
            - <NetID>-screenshot1.jpg
            - <NetID>-screenshot2.jpg
        - <NetID>-AppSec-Lab3.md


## Frequently Asked Questions

Kubernetes is a fairly complicated beast. To help you get oriented, we've created a [Frequently Asked Questions](FAQ.md) document that should help with common questions. As, always, please make use of office hours and ask questions by email when you run into trouble!

## Part 0: Setting up Your Environment
### 1) Synchronize Your Repository and Acquire the Lab Material
---
Log into GitHub within any web browser and create an empty, **private** repository named ``<NetID>-appsec3``.
```
cd ~
git clone https://github.com/NYUJRA/AppSec3.git AppSec3
cd AppSec3
git remote remove origin
git init
git remote add origin https://<YourGitHubHandle>:<YourPersonalAccessToken>@github.com/<YourGitHubHandle>/<NetID>-appsec3.git
git push -u origin main
```


You should now have a local working directory in ``~/AppSec3`` that is configured to use your remote GitHub repository at ``https://github.com/<YourGitHubHandle>/<NetID>-appsec3`` as a version control system.

This assignment requires Docker, minikube, and kubectl. These are all installed on your
NYU-AppSec VM for the class. There is an install script included in this repository for
reference only. If you decide to perform this assignment outside your VM, you will be
responsible for troubleshooting any issues yourself. It should be stated that kubernetes can be confusing, so it is critical that students take the time with the commands and read the kubernetes documentation in order to troubleshoot issues.
```
bash nyu-appsec-a3-ubuntu20043lts-setup.sh
```

### 2) Rundown of Files

This repository has a lot of files. The following are files you will likely be
modifying throughout this assignment.

* Baselines/ - CIS Benchmarks 
* GiftcardSite/GiftcardSite/settings.py
* GiftcardSite/LegacySite/views.py
* GiftcardSite/k8/ - Giftcard site kubernetes files
* db/Dockerfile
* db/setup.sql - Database seed file
* db/k8/ - Database kubernetes files
* proxy/Dockerfile
* proxy/k8/ - Proxy kubernetes files


### 3) Getting it to Work 

Once you have installed the necessary software, you are ready to run the whole thing
using minikube. First, start minikube.

```
minikube start
```

You will also need to set things up so that docker will use minikube, by running:

```
eval $(minikube docker-env)
```

Next,  we need to build the Dockerfiles Kubernetes will use to create the
cluster. This can be done using the following lines, assuming you are in the
root directory of the repository.

```
docker build -t nyuappsec/assign3:v0 .
docker build -t nyuappsec/assign3-proxy:v0 proxy/
docker build -t nyuappsec/assign3-db:v0 db/
```

Then use kubectl to create the pods and services needed for our project. Again,
these commands assume you are in the root directory of the repository.

```
kubectl apply -f db/k8
kubectl apply -f GiftcardSite/k8
kubectl apply -f proxy/k8
```
Verify that the pods and services were created correctly.

```
kubectl get pods
kubectl get service
```

There should be three pod entries:

* One that starts with assignment3-django-deploy
* One that starts with mysql-container
* One that starts with proxy

They should each have status RUNNING after approximately a minute.

There should also be four service entries:

* One called kubernetes
* One called assignment3-django-service
* One called mysql-service
* One called proxy-service

To see if you can connect to the site, run the following command:

```
minikube service proxy-service
```

This should open your browser to the deployed site. You should be able to view
the first page of the site, and navigate around. If this worked, you are ready
to move on to the next part.

## Part 1: Remediate Security Review Findings

The security team at your organization assessed the application deployment
against a subset of security baselines and found that it failed most 
controls. Unfortunately for you, this applicaiton is a high priority, and you 
have been charged with remediating all the hits of the security review before 
deployment of the applicaiton. The [`SecurityReview`](https://github.com/NYUJRA/AppSec3/tree/master/SecurityReview) directory contains the
controls, control number, results and remediation for each control. Additional
information, and audit methods are available in the corresponding CIS Benchmarks
in the [`Benchmarks`](https://github.com/NYUJRA/AppSec3/tree/master/Benchmarks) 
directory. It is important to research source documentation 
on proper implementation of the security controls, and perform testing to ensure
the proper functionality of the application. Careful documentation of all 
modifications to the application and configurations in order to implement each 
control is critical for maintainability of the application and is requried for
full credit.

###  Sub Task A) Validate findings (1 pt each)

The security team has done their best to review the application and provide guidance
on how to meet the security controls. They do not have a comprehensive knowledge of
the application and could have made some mistakes in their review. For each control
use the audit guide in the [`Benchmarks`](https://github.com/NYUJRA/AppSec3/tree/master/Benchmarks)
directory to validate the findings of the security team. Take a screenshot of the
result and document the steps you took

### Sub Task B) Remediate (1 pt each)

The security team gave remediation guidance for each failing control. Its your job
to implement the remediation. This will requrie you to understand the controls 
intention as well as the technology used to implement the control. Document the 
steps you took to implement the control. Use ```git``` to ensure that the changes you
make to any files are reflected in you repository

### Sub Task C) Verify finding resolution (1 pt each)

After each remediation, rebuild the affected container or reapply the affected
kubernetes configuration and verify that the control is now passing per the audit
guide. Take a screenshot of the result and document as necessary. 

## Part 2: (40 pts)

Validation of security controls can be a huge overhead to an organization if done 
manually. Choose two of the controls to check on an hourly basis. One of the
controls must check a value in the database (Oracle MySQL 8.0 :: 2.7, 2.9, or 4.2)
This automated check should be implemented using Kubernetes Jobs which you can read
about [here](https://kubernetes.io/docs/concepts/workloads/controllers/job/). Again,
you must document in your report, take screenshots of your output, and commit all
files to your repo for grading.


## Concluding Remarks

### Points breakdown
Part 1 has 20 controls that need validated, remediated, and verified. Those steps are worth 1pt a piece (20+20+20 = 60 total). Part 2 is two tasks worth 20pts a piece (20 + 20 = 40) for a total of 100 pts

### More (optional) fun
With the changes you made in this assignment, your company is a lot closer to a
decent deployment solution. However, even with the changes, there are a lot of
things that are still lacking.

One of the benefits of using Kubernetes is the ability to create replicas that
are load balanced to avoid overwhelming one instance of the application. The
same can be done with other micro-services such as the database, though this
would require database syncing across the difference database instances. These
solutions do not currently exist in this version of the assignment.

For more experience working with cloud security and deployment, consider taking
this one step further and replicating these micro-services. Attempt to load
balance over many replicas, and syncing databases. Try adding logging
and other useful tools.

Though these attempts will not be graded, and should not be submitted as part of
the assignment, they should help you learn a lot about how using cloud
deployment helps you preserve the availability of your service (and the
micro-services that comprise it) and how good monitoring and logging can help you
spot errors in the application before they become serious issues.
