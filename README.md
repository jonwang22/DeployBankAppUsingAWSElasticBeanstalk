# Deploying a Banking Application Using AWS Elastic Beanstalk
## Purpose

I wanted to learn more about how to use Jenkins to build and test code and AWS Elastic Beanstalk to deploy and host my web application.
Jenkins is a CI/CD tool that allows us to build and test our code hosted here in this Github repository.  
AWS Elastic Beanstalk is a managed service that eliminates our need to manually handle capacity provisioning, load balancing, auto-scaling, and application health monitoring.  
We can simply just deploy our app and be ready to go for our customers.

The steps listed below show what was done to get this web application up and running.

## Steps Taken to Implement

1. Git clone code repository to personal repository without creating a fork. The reason why we don't want to fork is because we want to have a completely independent copy of the repository, unrelated to the original repository. We can also modify access controls or repository settings that would not be possible if we had forked the original repository.

      a. You can do it multiple ways but I chose to use this method https://gist.github.com/hohowin/954fba73f5a02d37e15a6ea5e5b10b54
         - Create empty repo in Github, cannot have any commits.
         - On EC2 instance or local machine, create empty directory and run `git init`
         - Navigate to that empty directory and run `git pull $SOURCE_GITHUB_REPO` for your code
         - Once the source code has been pulled down to your directory, run `git remote add origin $GIT_URL_NEW_CREATED_REPO` and then `git push origin main\master`
            * Git remote essentially creates a link between your local respository to your remote repository hosted in Github.
      b. The other ways you can clone a repository:
         - Git clone source repository and add a remote link to destination repository.
         - Git clone both source and destination repositories, copy files locally from source repo to destination repo and then git push files in the destination repo.
   
2. Created EC2 Instance to host Jenkins

      a. Created EC2 Instance in AWS.
      b. Created a security group that allowed SSH, HTTP and custom port 8080 for Jenkins.

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

4. Logged in to Jenkins and created an admin user for me to perform the build and test actions I need before deploying my code to production via AWS Elastic Beanstalk.

5. Created a new multibranch pipeline and linked it to my github repo here so that Jenkins knows what source code it needs to build out and test.

6. Build out the source code and test it. Jenkins completed the build and it went through multiple stages during the build.

   	a. First stage is Checkout SCM. This stage of Jenkins is cloning and fetching the git repository to begin building it.

   	b. Second stage is Build. This stage is where Jenkins downloads and installs dependencies to make sure it has what it needs to test the logical code in the package.

   	c. Third stage is Test. This stage is where the logic is tested from the application. Are the functions written and features written in the application working properly with the unit tests that were written for the application.

   	d. Issues can arise during the build and testing phase. Build issues can be caused by configuration drift or version diffs on dependencies. Testing issues could happen if the unit tests weren't written properly or the logic doesn't work. Using Jenkins allows developers to iterate and test their code many times as needed before shipping it out to production environments, ensuring a working product/service for the customers to use. 

7. Created AWS Elastic Beanstalk Environment to deploy our application.

      a. This allowed us to have our application managed by AWS Elastic Beanstalk. We don't have to worry about capacity, load balancing, autoscaling, or application health monitoring.

      b. We can choose to create a web server environment or a worker environment and choose which platform we want to use. For this app we used Python.


## System Design Diagram



## Issues/Troubleshooting

1. Git Cloning a Repo without Forking
2. Creating AWS Elastic Beanstalk environment. Health Monitoring settings. Default options selected, needed to use Basic and had to deselect a managed monitoring service in order to proceed and create the environment.
3. Unable to access the application via the domain.
Error Log:
```
2024/07/26 15:19:29 [error] 2753#2753: *5 connect() failed (111: Connection refused) while connecting to upstream, client: 96.255.240.58, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8000/", host: "deploybankappusingawselasticbean-env-1.eba-h9au3tz6.us-east-1.elasticbeanstalk.com"
2024/07/26 15:19:29 [error] 2753#2753: *5 connect() failed (111: Connection refused) while connecting to upstream, client: 96.255.240.58, server: , request: "GET /favicon.ico HTTP/1.1", upstream: "http://127.0.0.1:8000/favicon.ico", host: "deploybankappusingawselasticbean-env-1.eba-h9au3tz6.us-east-1.elasticbeanstalk.com", referrer: "http://deploybankappusingawselasticbean-env-1.eba-h9au3tz6.us-east-1.elasticbeanstalk.com/"
```

Found ElasticBeanstalk Error logs. Web Server isn't starting up.
```
Jul 28 02:29:37 ip-172-31-47-211 web: Traceback (most recent call last):
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/arbiter.py", line 609, in spawn_worker
Jul 28 02:29:37 ip-172-31-47-211 web: worker.init_process()
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/workers/gthread.py", line 95, in init_process
Jul 28 02:29:37 ip-172-31-47-211 web: super().init_process()
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/workers/base.py", line 134, in init_process
Jul 28 02:29:37 ip-172-31-47-211 web: self.load_wsgi()
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/workers/base.py", line 146, in load_wsgi
Jul 28 02:29:37 ip-172-31-47-211 web: self.wsgi = self.app.wsgi()
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/app/base.py", line 67, in wsgi
Jul 28 02:29:37 ip-172-31-47-211 web: self.callable = self.load()
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/app/wsgiapp.py", line 58, in load
Jul 28 02:29:37 ip-172-31-47-211 web: return self.load_wsgiapp()
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/app/wsgiapp.py", line 48, in load_wsgiapp
Jul 28 02:29:37 ip-172-31-47-211 web: return util.import_app(self.app_uri)
Jul 28 02:29:37 ip-172-31-47-211 web: File "/var/app/venv/staging-LQM1lest/lib64/python3.7/site-packages/gunicorn/util.py", line 371, in import_app
Jul 28 02:29:37 ip-172-31-47-211 web: mod = importlib.import_module(module)
Jul 28 02:29:37 ip-172-31-47-211 web: File "/usr/lib64/python3.7/importlib/__init__.py", line 127, in import_module
Jul 28 02:29:37 ip-172-31-47-211 web: return _bootstrap._gcd_import(name[level:], package, level)
Jul 28 02:29:37 ip-172-31-47-211 web: File "<frozen importlib._bootstrap>", line 1006, in _gcd_import
Jul 28 02:29:37 ip-172-31-47-211 web: File "<frozen importlib._bootstrap>", line 983, in _find_and_load
Jul 28 02:29:37 ip-172-31-47-211 web: File "<frozen importlib._bootstrap>", line 965, in _find_and_load_unlocked
Jul 28 02:29:37 ip-172-31-47-211 web: ModuleNotFoundError: No module named 'application'
Jul 28 02:29:37 ip-172-31-47-211 web: [2024-07-28 02:29:37 +0000] [2805] [INFO] Worker exiting (pid: 2805)
Jul 28 02:29:37 ip-172-31-47-211 web: [2024-07-28 02:29:37 +0000] [2798] [ERROR] Worker (pid:2805) exited with code 3
Jul 28 02:29:37 ip-172-31-47-211 web: [2024-07-28 02:29:37 +0000] [2798] [ERROR] Shutting down: Master
Jul 28 02:29:37 ip-172-31-47-211 web: [2024-07-28 02:29:37 +0000] [2798] [ERROR] Reason: Worker failed to boot.
```
The reason why we were getting these errors is because when we uploaded our application, we uploaded the zip folder directly downloaded from GitHub. AWS Elastic Beanstalk looks for the application file at the top level when the zip file is unzipped. In order to provide that properly, I had to take the Github zip and unzip it to get the main folder. Go into that folder and then zip all the files up inside and then upload that zip file to AWS Elastic Beanstalk. Then the web app could be accessed by the domain given.

## Optimization



## Conclusion
