### Objective
Create a Flask Docker Image and have AWS host the Docker Container for us in the cloud. 

### Running Flask app locally 
1. cd to project directory
2. `.\venv\Scripts\activate` to start virtual env
3. `pip install --upgrade pip` 
4. `pip install -r .\requirements.txt` to install dependencies 
5. `python .\app.py ` to start the webserver on port 5000
6. Go to `http://localhost:5000/` to confirm you are seeing 'Hello World!'
7. `Ctrl + C` to stop web server
8. `deactivate` to stop the venv

### Building Docker Image 
1. cd to project directory
2. Run cmd `docker image build -t flask-demo-vs .`
3. `docker run -p 5000:5000 -d flask-demo-vs` to run the image (which is now called a Container). Note we are binding port 5000 (host) to port 5000 on the container. `[host_port]:[container_port]`
4. Go to http://localhost:5000/ on local machine to make sure container is running. 
5. Note the long string of chars, like `d1eaa386d55db5ca1298e5931906bb967c1dbff87d800dc8bb841f424ea50f1f` - this is the container id. 
6. Run command: `docker stop d1eaa386d55db5ca1298e5931906bb967c1dbff87d800dc8bb841f424ea50f1f` to stop the container. 
7. Run command: `docker system prune` to clean up resources after stopping container. 
8. Run command: `docker image ls` to see all images on the machine.

### Setup AWS CLI for uploading Docker Image
* https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* Go into AWS Console > IAM dashboard 
* Users > Add users > name it
* Select programmatic access to get an access key id + secret 
* Click Attach existing policies directly 
* Choose ECS_FullAccess and AmazonEC2ContainerRegistryFullAccess
* Tag: name == ecr-and-ecs-full-access
* Create user and copy the access key ID + secret access key (like AKIA4HNKMWVR2UBBDVPF and MIb7JUYmKM5ZVfYSgZsSoWPJgB9bp5vkNzGtlo7R)
* Open terminal/powershell after installing the AWS CLI 
* Run cmd: `aws configure`, then enter the access key + secret access key when prompted 
* us-east-1 is a common default region for US 

### Uploading Docker Image to ECR in AWS 
1. AWS Console > ECR (Elastic Container Registry) 
2. Create a repo 
3. Name the repo, like `flask-demo-vs` (it's nice to be consistent with your Docker Image name)
4. Click on the repo name after it's created 
5. Click View Push Commands 
6. Run the commands from the project dir
7. Confirm you can see the docker image in ECR repo after upload.
8. Copy the Image URI from ECR, it will look like `840560325987.dkr.ecr.us-east-1.amazonaws.com/flask-demo-vs:latest`

### Hosting the Docker Container with ECS in AWS 
1. AWS Console > ECS > Create Cluster
2. EC2 Linux + Networking
3. Name cluster: flask-cluster
4. t2.micro is for free-tier 
5. Choose the VPC that already exists for your AWS account 
6. Choose the Subnet 
7. Auto Assign public IP == enabled 
8. Create a new security group 
9. Click Create 
10. It will take some time for the resources to get made. Wait for it to finish. 
11. Click View Cluster after it's done 
12. Click Tasks tab 
13. Create new Task Definition, choose EC2 
14. Name: EcsFlaskDemo
15. Task size: 512 MiB and 1 VCPU
16. Add Container > Container Name: FlaskContainerTest
17. Image: paste in the URI from ECR (`840560325987.dkr.ecr.us-east-1.amazonaws.com/flask-demo-vs:latest`)
19. Add Port mappings: Can do 5000:5000 
20. Click Add 
21. Click Create 
22. Go back to view tasks > run the task you just made (Choose EC2)
23. Go to EC2 dashboard 
24. Click on the running EC2 instance
25. Copy the public IPv4 address, like 35.171.26.253
26. Go to Security > Click on Security Group 
27. Edit Inbound Rules 
28. Allow all traffic IPv4 and IPv6 on port 5000
29. Save rule 
30. Now try to reach your web server by hitting port 5000, like this: 35.171.26.253:5000


### Appendix 

If you get the error: "Hardware assisted virtualization and data execution protection must be enabled in the BIOS" - try running this cmd and restarting Windows: `dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All`. If that doesn't work, go into BIOS and make sure Virtualization or SVM (Secure Virtual Machine) option is enabled.

https://stackoverflow.com/questions/39684974/docker-for-windows-error-hardware-assisted-virtualization-and-data-execution-p

