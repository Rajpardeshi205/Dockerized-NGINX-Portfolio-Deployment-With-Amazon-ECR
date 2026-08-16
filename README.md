# Mini Project: Dockerized NGINX Portfolio Deployment With Amazon ECR

# Introduction

This mini project demonstrates how to containerize a static portfolio website using Docker, optimize the image with Alpine Linux, store the image securely in Amazon Elastic Container Registry (Amazon ECR), and deploy it using NGINX. The project also uses Docker Bind Mounts and Named Volumes to separate application files from container data, making the deployment portable and easy to manage.

# Project Summary

In this project, I:

- Installed and configured Docker on an Amazon EC2 instance.
- Built a lightweight Docker image using Alpine Linux and NGINX.
- Created a Dockerfile to package the portfolio website.
- Configured AWS CLI authentication.
- Created a private Amazon ECR repository.
- Tagged and pushed the Docker image to Amazon ECR.
- Created a Docker Named Volume for NGINX logs.
- Used a Bind Mount to serve website files from the EC2 instance.
- Configured NGINX inside the container.
- Successfully hosted the portfolio website through Docker.

# Architecture

![ChatGPT Image Aug 6, 2026, 10_38_18 PM.png](ChatGPT_Image_Aug_6_2026_10_38_18_PM.png)

# Implementation Step

## 1. Update & Install Dependency

```jsx
 sudo yum update
 sudo yum install docker -y
```

## 2. Start Docker

```jsx
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

## 3. Add User To Docker Group

```jsx
sudo gpasswd -a ec2-user docker
```

## 4. Change /var/lib/docker Permission

```jsx
cd /var/lib/
```

![image.png](image.png)

```jsx
sudo chmod 770 docker/
sudo chgrp -R docker docker
cd docker/
```

![image.png](image%201.png)

## 5. Create Portfolio Folder In Home Directory

```jsx
mkdir Portfolio
```

![image.png](image%202.png)

## 6. Create IAM User

Attach Policy

- [`AWSAppRunnerServicePolicyForECRAccess](https://us-east-1.console.aws.amazon.com/iam/home?region=ap-south-1#/policies/details/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2Fservice-role%2FAWSAppRunnerServicePolicyForECRAccess)`
- [`EC2InstanceProfileForImageBuilderECRContainerBuilds](https://us-east-1.console.aws.amazon.com/iam/home?region=ap-south-1#/policies/details/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FEC2InstanceProfileForImageBuilderECRContainerBuilds)`

Create Access Key (CLI)

```jsx
Access Key :
AKIAYU47RJVVM4RYBROY

Secret Access Key :
Gb1LQsa****************************8
```

## 7. AWS Configuration

```jsx
aws configure
```

![image.png](image%203.png)

## 8. Create Docker File

```jsx
vim Dockerfile
```

```jsx
FROM alpine:latest
LABEL author="Raj"
RUN apk update && apk add nginx
WORKDIR /usr/share/nginx/html/
COPY README.md .
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```

![image.png](image%204.png)

## 9. Build Docker Image

```jsx
 docker build -t my-port .
 docker images
```

![image.png](image%205.png)

## 10. Go To Amazon ERC

Create New Repo

![image.png](image%206.png)

Name : `raj/my-port`

![image.png](image%207.png)

![image.png](image%208.png)

Copy & Use Commands

```jsx
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 594650680682.dkr.ecr.us-east-1.amazonaws.com
```

![image.png](image%209.png)

```jsx
docker tag raj/my-port:latest 594650680682.dkr.ecr.us-east-1.amazonaws.com/raj/my-port:latest
```

![image.png](image%2010.png)

```jsx
docker push 594650680682.dkr.ecr.us-east-1.amazonaws.com/raj/my-port:latest
```

![image.png](image%2011.png)

![image.png](image%2012.png)

## 11. Create Named Volume

```jsx
docker volume create my-vol
docker volume ls
```

![image.png](image%2013.png)

## 12. Attached To Container Using Bind Mount & Volume

```jsx
docker run -d -p 80:80 -v ~/Portfolio:/usr/share/nginx/html -v my-vol:/var/log/nginx --name my-port my-port
```

![image.png](image%2014.png)

## 13. Go Inside Container

```jsx
docker exec -it my-port sh
```

![image.png](image%2015.png)

## 14. Configure Nginx

```jsx
cd / etc / nginx / http.d;
```

```jsx
apk update
apk add vim
```

```jsx
vim default.conf
```

```jsx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /usr/share/nginx/html;
        index index.html;

        # Everything is a 404
        location / {
                try_files $uri $uri/ =404;
        }
}
```

![image.png](image%2016.png)

## 15. Check On Browser

Restart Container

```jsx
 docker restart my-port
```

Copy Public IP

```jsx
54.87.169.40
```

![image.png](image%2017.png)

# Problems Faced & Solutions

### 1. Docker Permission Denied

After installing Docker, the `ec2-user` could not execute Docker commands and received a permission denied error while connecting to the Docker daemon.

```jsx
Added the ec2-user to the Docker group and reconnected to the EC2 instance so Docker commands could be executed without using sudo.

sudo gpasswd -a ec2-user docker
```

### 2. Docker Directory Permission Issue

Docker did not have the required permissions to access files under `/var/lib/docker`, causing permission-related issues.

```jsx
Updated the ownership and permissions of the Docker storage directory so members of the Docker group could access it.

sudo chmod 770 /var/lib/docker
sudo chgrp -R docker /var/lib/docker
```

### 3. NGINX Returned 404 Not Found

After running the container, the website displayed a **404 Not Found** error because of the default NGINX configuration.

```jsx
Modified the NGINX configuration to serve files from the correct document root and restarted the container.

location / {
    try_files $uri $uri/ =404;
}
```

# Summary

This mini project successfully demonstrated the complete workflow of containerizing and deploying a static portfolio website using **Docker**, **NGINX**, and **Amazon Elastic Container Registry (ECR)** on an **AWS EC2** instance. A lightweight Docker image was built using **Alpine Linux**, reducing image size while maintaining efficient performance. The image was securely stored in Amazon ECR and deployed as a container on EC2.

Throughout the implementation, Docker **Bind Mounts** and **Named Volumes** were used to separate website content and log data from the container, ensuring easier maintenance and data persistence. Several real-world challenges—including Docker permission issues, Amazon ECR authentication, image tagging errors, NGINX configuration problems, and data persistence—were identified and resolved through systematic troubleshooting.

Overall, this project strengthened practical knowledge of **Docker containerization, AWS ECR, Linux administration, NGINX configuration, Docker storage management, and cloud-based application deployment**. It also demonstrated industry-standard practices for building lightweight, portable, and scalable containerized applications suitable for modern DevOps workflows.
