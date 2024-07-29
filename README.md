# Deploying a Banking Application Using AWS Elastic Beanstalk
## Purpose

I wanted to learn more about how to use Jenkins to build and test code and AWS Elastic Beanstalk to deploy and host my web application.
Jenkins is a CI/CD tool that allows us to build and test our code hosted here in this Github repository.  
AWS Elastic Beanstalk is a managed service that eliminates our need to manually handle capacity provisioning, load balancing, auto-scaling, and application health monitoring.  
We can simply just deploy our app and be ready to go for our customers.

The steps listed below show what was done to get this web application up and running.

## Steps Taken to Implement


1. Git clone code repository to personal repository without creating a fork. The reason why we don't want to fork is because we want to have a completely independent copy of the repository, unrelated to the original repository. We can also modify access controls or repository settings that would not be possible if we had forked the original repository.

   * You can do it multiple ways but I chose to use this method https://gist.github.com/hohowin/954fba73f5a02d37e15a6ea5e5b10b54
      - Create empty repo in Github, cannot have any commits.
      - On EC2 instance or local machine, create empty directory and run `git init`
      - Navigate to that empty directory and run `git pull $SOURCE_GITHUB_REPO` for your code
      - Once the source code has been pulled down to your directory, run `git remote add origin $GIT_URL_NEW_CREATED_REPO` and then `git push origin main\master`
        * Git remote essentially creates a link between your local respository to your remote repository hosted in Github.
        
   * The other ways you can clone a repository:
      - Git clone source repository and add a remote link to destination repository.
      - Git clone both source and destination repositories, copy files locally from source repo to destination repo and then git push files in the destination repo.
   
2. Created EC2 Instance to host Jenkins
   
   * Created EC2 Instance in AWS.
   
   * Created a security group that allowed SSH, HTTP and custom port 8080 for Jenkins.

3. Installed Jenkins to my Jenkins server and started it up.

   ```
   # Updating existing packages on system, installing fontconfig, java runtime env, software properties common to manage additional software repos. Added deadsnakes PPA and installed python3.7
   sudo apt update && sudo apt install fontconfig openjdk-17-jre software-properties-common && sudo add-apt-repository ppa:deadsnakes/ppa && sudo apt install python3.7 python3.7-venv

   # Downloaded the Jenkins respository key. Added the key to the /usr/share/keyrings directory
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

   # Added Jenkins repo to sources list
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

   # Downloaded all updates for packages again, installed Jenkins
   sudo apt-get update
   sudo apt-get install jenkins

   # Started Jenkins and checked to make sure Jenkins is active and running with no issues
   sudo systemctl start jenkins
   sudo systemctl status jenkins
   ```

4. Interact with Jenkins

   * Connected to Jenkins Server via web browser using IP Address and port 8080
   * Created Admin user account
   * Created Multibranch Pipeline within Jenkins
   * Linked this github repo to Jenkins
   * Started a build.

5. Jenkins Build/Test

   * First stage is Checkout SCM. This stage of Jenkins is cloning and fetching the git repository to begin building it.
     ![Screenshot 2024-07-26 at 2 03 23 AM](https://github.com/user-attachments/assets/b1559658-49b0-448b-ad56-19ff48e73d73)

   * Second stage is Build. This stage is where Jenkins downloads and installs dependencies to make sure it has what it needs to test the logical code in the package.
     ![Screenshot 2024-07-26 at 2 03 34 AM](https://github.com/user-attachments/assets/a2127b33-9aa7-4b48-94d4-b61ef3e30355)
     ![Screenshot 2024-07-26 at 2 03 59 AM](https://github.com/user-attachments/assets/2f5ff853-afbb-4460-a5e9-ef23e57750da)

   * Third stage is Test. This stage is where the logic is tested from the application. Are the functions written and features written in the application working properly with the unit tests that were written for the application.
     ![Screenshot 2024-07-26 at 2 04 10 AM](https://github.com/user-attachments/assets/7cac9b12-607e-40b5-a60e-7a2b75187a05)

   * Issues can arise during the build and testing phase. Build issues can be caused by configuration drift or version diffs on dependencies. Testing issues could happen if the unit tests weren't written properly or the logic doesn't work. Using Jenkins allows developers to iterate and test their code many times as needed before shipping it out to production environments, ensuring a working product/service for the customers to use.
  
   * Successful Build
     ![Screenshot 2024-07-26 at 1 03 30 AM](https://github.com/user-attachments/assets/87340d5d-8bb0-4291-b661-492a1ad0524c) 

6. Deploy Application Using AWS Elastic Beanstalk

   * I needed to create Service Roles for my Elastic Beanstalk environment so that the service has permissions to perform the actions it needs to manage my application for me. Service Roles and their policies are listed below.
     * aws-elasticbeanstalk-service-role
       * AWSElasticBeanstalkWebTier
       * AWSElasticBeanstalkWorkerTier
       * AWSElasticBeanstalkMulticontainerDocker
     * Elastic-EC2
       * Allow EC2 to call AWS Services

   * Once I have my service roles, I need to create my Elastic Beanstalk Environment. This environment is what allows me to set up the environment my Application will be hosted on.

   * Environment Configuration
     * Select "Web Server Environment"
     * Enter Application name (I used DeployBankAppUsingAWSElasticBeanstalk)
     * For Managed Platform, select Python3.7
     * Upload the code from local. This is where you download the zip file from Github repo, unzip the folder, go into the folder and then zip the contents of that folder. Once you have the files zipped without the top level folder, then you can upload that zip file to Elastic Beanstalk. (See Issues/Troubleshooting for what happens if you upload the zip file directly after downloading it from Github)
     * Selected instance configuration
     * Selected VPC and subnet
     * Selected Root Volume type, used General Purpose (SSD) and assigned 10GB
     * Instance class just used free tier ones (t3.micro)
     * Selected Basic Health Monitoring, de-selected Managed Monitoring option
     * Once environment is built and ready, clicked the domain link to access the web application.

   * For more information about AWS Elastic Beanstalk, https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html

## System Design Diagram



## Issues/Troubleshooting

1. Git cloning Source Repo (Original repo) and adding remote link to Destination Repo (This one)

	I had some issues trying to figure out how to add another remote link after git cloning a repo to my local machine. I ended up dropping this and finding an easier way to pull the files and upload them to my new repo. See Steps above.

2. Creating AWS Elastic Beanstalk environment. Health Monitoring settings.

	After selecting Basic Health Monitoring, default selection for Managed Platform Updates settings was checked, in order to create the environment with Basic Health monitoring, you have to deselect the Managed Platform Updates selection.

3. Unable to access the application via the domain after Environment successfully created. Receiving Server 502 Error from nginx.

/var/log/nginx/error.log:
```
2024/07/26 15:19:29 [error] 2753#2753: *5 connect() failed (111: Connection refused) while connecting to upstream, client: X.X.X.X, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8000/", host: "deploybankappusingawselasticbean-env-1.eba-h9au3tz6.us-east-1.elasticbeanstalk.com"
2024/07/26 15:19:29 [error] 2753#2753: *5 connect() failed (111: Connection refused) while connecting to upstream, client: X.X.X.X, server: , request: "GET /favicon.ico HTTP/1.1", upstream: "http://127.0.0.1:8000/favicon.ico", host: "deploybankappusingawselasticbean-env-1.eba-h9au3tz6.us-east-1.elasticbeanstalk.com", referrer: "http://deploybankappusingawselasticbean-env-1.eba-h9au3tz6.us-east-1.elasticbeanstalk.com/"
```

Found ElasticBeanstalk Error logs. Web Server isn't starting up.
```
Jul 28 02:29:37 ip-X-X-X-X web: Traceback (most recent call last):
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/arbiter.py", line 609, in spawn_worker
Jul 28 02:29:37 ip-X-X-X-X web: worker.init_process()
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/workers/gthread.py", line 95, in init_process
Jul 28 02:29:37 ip-X-X-X-X web: super().init_process()
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/workers/base.py", line 134, in init_process
Jul 28 02:29:37 ip-X-X-X-X web: self.load_wsgi()
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/workers/base.py", line 146, in load_wsgi
Jul 28 02:29:37 ip-X-X-X-X web: self.wsgi = self.app.wsgi()
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/app/base.py", line 67, in wsgi
Jul 28 02:29:37 ip-X-X-X-X web: self.callable = self.load()
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/app/wsgiapp.py", line 58, in load
Jul 28 02:29:37 ip-X-X-X-X web: return self.load_wsgiapp()
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/app/wsgiapp.py", line 48, in load_wsgiapp
Jul 28 02:29:37 ip-X-X-X-X web: return util.import_app(self.app_uri)
Jul 28 02:29:37 ip-X-X-X-X web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/util.py", line 371, in import_app
Jul 28 02:29:37 ip-X-X-X-X web: mod = importlib.import_module(module)
Jul 28 02:29:37 ip-X-X-X-X web: File "/usr/lib64/python3.7/importlib/__init__.py", line 127, in import_module
Jul 28 02:29:37 ip-X-X-X-X web: return _bootstrap._gcd_import(name[level:], package, level)
Jul 28 02:29:37 ip-X-X-X-X web: File "<frozen importlib._bootstrap>", line 1006, in _gcd_import
Jul 28 02:29:37 ip-X-X-X-X web: File "<frozen importlib._bootstrap>", line 983, in _find_and_load
Jul 28 02:29:37 ip-X-X-X-X web: File "<frozen importlib._bootstrap>", line 965, in _find_and_load_unlocked
Jul 28 02:29:37 ip-X-X-X-X web: ModuleNotFoundError: No module named 'application'
Jul 28 02:29:37 ip-X-X-X-X web: [2024-07-28 02:29:37 +0000] [2805] [INFO] Worker exiting (pid: 2805)
Jul 28 02:29:37 ip-X-X-X-X web: [2024-07-28 02:29:37 +0000] [2798] [ERROR] Worker (pid:2805) exited with code 3
Jul 28 02:29:37 ip-X-X-X-X web: [2024-07-28 02:29:37 +0000] [2798] [ERROR] Shutting down: Master
Jul 28 02:29:37 ip-X-X-X-X web: [2024-07-28 02:29:37 +0000] [2798] [ERROR] Reason: Worker failed to boot.
```
The reason why we were getting these errors is because when we uploaded our application, we uploaded the zip folder directly downloaded from GitHub. AWS Elastic Beanstalk looks for the application file at the top level when the zip file is unzipped. In order to provide that properly, I had to take the Github zip and unzip it to get the main folder. Go into that folder and then zip all the files up inside and then upload that zip file to AWS Elastic Beanstalk. Then the web app could be accessed by the domain given. This stack overflow post helped me figure this out. https://stackoverflow.com/questions/62479386/no-module-named-application-error-while-deploying-simple-web-app-to-elastic-be

There were many other rabbit holes I dug myself into that were not the root of the issue or had any relationship to the issue. List is below:

   * Application code designated port incorrect. (This was wrong, nothing was wrong with port 5000 stated in application code)
   * Security Group settings for inbound traffic on my App Server. (Nothing was wrong with the security group)
   * Procfile settings incorrect. (This wasn't the root of the issue but led me to the root) Doc on Procfile using Python https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/python-configuration-procfile.html

## Optimization

1. What are the benefits of using managed services for cloud infrastructure?

	* Only need to be responsible for application source code and building/testing the code in a tool like Jenkins.
 	* Managed Services handle underlying infrastructure for you. No need to worry about Capacity, Load Balancing, Auto-scaling, or health monitoring.
  	* No need to worry about manually configuring your environment. Managed services provides options for you to choose from and the service handles the rest.

2. What are some issues that a retail bank would face choosing this method of deployment and how would you address/resolve them?

	* Manual process from Build/Test to deploy. Maybe incorporate a tool such as Infrastructure as Code (IAC) depending on the cloud environment you're using.
 	* Version controlling application. Having to download the zip file from Github Repo then uploading it manually to Elastic Beanstalk could cause errors and issues. Automating this would be the best way to deal with reducing the impact and chances of human error.

3. What are other disadvantages of using elastic beanstalk or similar managed services for deploying applications?

	* Unable to control and customize certain aspects of the service.
 	* You're stuck with what they provide you. Python3.7 is on a deprecation path. Users of AWS Elastic Beanstalk will need to make sure they update their code and software to use Python3.8 and above. Platforms are limited in the sense of you can only use what the managed service provides.
  	* Potentially higher operational costs
   	* Dependent on new features or rollouts or service compatibility, have to wait on service provider for those updates.
   	* Unclear issues and logs. They may be less straightforward.

## Conclusion

This is a great project to figure out how to use Jenkins and deploy to AWS Elastic Beanstalk.
