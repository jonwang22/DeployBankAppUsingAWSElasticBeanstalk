# Deploying a Banking Application Using AWS Elastic Beanstalk
## Purpose

We want to deploy our banking application for others to use. To do this, we are leveraging Jenkins and AWS Elastic Beanstalk.  
Jenkins is our CI/CD tool that allows us to build and test our code hosted here in this Github repository.  
AWS Elastic Beanstalk is a managed service that reduces our need to handle capacity provisioning, load balancing, scaling, and application health monitoring.  
We can simply just deploy our app and be ready to go for our customers.

## Steps Taken to Implement

1. Cloned repository from another repository so that I can have full control over this repository and the files in it. If I need to make any sort of changes then I can.

2. Created EC2 Instance to host Jenkins for me to build and test my code.

3. Installed Jenkins to my Jenkins server and started it up.

4. Logged in to Jenkins and created an admin user for me to perform the build and test actions I need before deploying my code to production via AWS Elastic Beanstalk.

5. Created a new multibranch pipeline and linked it to my github repo here so that Jenkins knows what source code it needs to build out and test.

6. Build out the source code and test it. Jenkins completed the build and it went through multiple stages during the build.

   	a. First stage is Checkout SCM. This stage of Jenkins is cloning and fetching the git repository to begin building it.

   	b. Second stage is Build. This stage is where Jenkins downloads and installs dependencies to make sure it has what it needs to test the logical code in the package.

   	c. Third stage is Test. This stage is where the logic is tested from the application. Are the functions written and features written in the application working properly with the unit tests that were written for the application.

   	d. Issues can arise during the build and testing phase. Build issues can be caused by configuration drift or version diffs on dependencies. Testing issues could happen if the unit tests weren't written properly or the logic doesn't work. Using Jenkins allows developers to iterate and test their code many times as needed before shipping it out to production environments, ensuring a working product/service for the customers to use. 

7. Created AWS Elastic Beanstalk Environment to deploy our application.


## System Design Diagram



## Issues/Troubleshooting



## Optimization



## Conclusion
