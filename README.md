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


## Optimization



## Conclusion
