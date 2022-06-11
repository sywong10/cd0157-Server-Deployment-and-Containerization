# Deploying a Flask API

This is the project starter repo for the course Server Deployment, Containerization, and Testing.

In this project you will containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app that will be used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'. 
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token. 

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.



## Prerequisites

* Docker Desktop - Installation instructions for all OSes can be found <a href="https://docs.docker.com/install/" target="_blank">here</a>.
* Git: <a href="https://git-scm.com/downloads" target="_blank">Download and install Git</a> for your system. 
* Code editor: You can <a href="https://code.visualstudio.com/download" target="_blank">download and install VS code</a> here.
* AWS Account
* Python version between 3.7 and 3.9. Check the current version using:
```bash
#  Mac/Linux/Windows 
python --version
```
You can download a specific release version from <a href="https://www.python.org/downloads/" target="_blank">here</a>.

* Python package manager - PIP 19.x or higher. PIP is already installed in Python 3 >=3.4 downloaded from python.org . However, you can upgrade to a specific version, say 20.2.3, using the command:
```bash
#  Mac/Linux/Windows Check the current version
pip --version
# Mac/Linux
pip install --upgrade pip==20.2.3
# Windows
python -m pip install --upgrade pip==20.2.3
```
* Terminal
   * Mac/Linux users can use the default terminal.
   * Windows users can use either the GitBash terminal or WSL. 
* Command line utilities:
  * AWS CLI installed and configured using the `aws configure` command. Another important configuration is the region. Do not use the us-east-1 because the cluster creation may fails mostly in us-east-1. Let's change the default region to:
  ```bash
  aws configure set region us-east-2  
  ```
  Ensure to create all your resources in a single region. 
  * EKSCTL installed in your system. Follow the instructions [available here](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl) or <a href="https://eksctl.io/introduction/#installation" target="_blank">here</a> to download and install `eksctl` utility. 
  * The KUBECTL installed in your system. Installation instructions for kubectl can be found <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/" target="_blank">here</a>. 


## Initial setup

1. Fork the <a href="https://github.com/udacity/cd0157-Server-Deployment-and-Containerization" target="_blank">Server and Deployment Containerization Github repo</a> to your Github account.
1. Locally clone your forked version to begin working on the project.
```bash
git clone https://github.com/sywong10/cd0157-Server-Deployment-and-Containerization.git
cd cd0157-Server-Deployment-and-Containerization/
```
1. These are the files relevant for the current project:
```bash
.
├── Dockerfile 
├── README.md
├── aws-auth-patch.yml #ToDo
├── buildspec.yml      #ToDo
├── ci-cd-codepipeline.cfn.yml #ToDo
├── iam-role-policy.json  #ToDo
├── main.py
├── requirements.txt
├── simple_jwt_api.yml
├── test_main.py  #ToDo
└── trust.json     #ToDo 
```

     
## Project Steps

Completing the project involves several steps:

1. Write a Dockerfile for a simple Flask API
2. Build and test the container locally
3. Create an EKS cluster
4. Store a secret using AWS Parameter Store
5. Create a CodePipeline pipeline triggered by GitHub checkins
6. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson.



## Running Container Locally

1. git clone the repo

2. 
$ docker build -t myimage .
$ docker image ls
$ docker run --name myContainer --env-file=.env_file -p 80:8080 myimage
$ docker container ls
$ docker ps
$ curl --request GET 'http://localhost:80'


get token:

endpoint / to get "healthy status":

$ curl --request GET 'http://localhost:80/'

use endpoint /auth to retrieve token:

$ export TOKEN=`curl --data '{"email":"abc@xyz.com","password":"mypwd"}' --header "Content-Type: application/json" -X POST localhost:80/auth | jq -r .token`
$ echo $TOKEN

endpoint /contents:

$ export TOKEN=`curl --data '{"email":"abc@xyz.com","password":"mypwd"}' --header "Content-Type: application/json" -X POST localhost:80/auth | jq -r .token`
$ curl --request GET 'http://localhost:8080/contents' -H "Authorization: Bearer ${TOKEN}" | jq .



## running app in EKS

github token: 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

http://a91feb16b26054ff29698b687dfbf376-1862089464.us-east-2.elb.amazonaws.com/


## setup EKS and code pipeline

git clone from https://github.com/sywong10/cd0157-Server-Deployment-and-Containerization.git

1. eksctl create cluster --name simple-jwt-api --region=us-east-2
2. aws sts get-caller-identity --query Account --output text    to get account number
3. replace <ACCOUNT_ID> with actual account ID in trust.json
4. aws iam create-role --role-name UdacityFlaskDeployCBKubectlRole --assume-role-policy-document file://trust.json --output text --query 'Role.Arn'
5. aws iam put-role-policy --role-name UdacityFlaskDeployCBKubectlRole --policy-name eks-describe --policy-document file://iam-role-policy.json
6. kubectl get -n kube-system configmap/aws-auth -o yaml > /tmp/aws-auth-patch.yml
7. add following to aws-auth-patch.yml
   mapRoles: |
   - groups:
     - system:masters
     rolearn: arn:aws:iam::<ACCOUNT_ID>:role/UdacityFlaskDeployCBKubectlRole
     username: build   
8. kubectl patch configmap/aws-auth -n kube-system --patch "$(cat aws-auth-patch.yml)"
9. aws ssm put-parameter --name JWT_SECRET --overwrite --value "myjwtsecret" --type SecureString
10. use ci-cd-codepipeline.cfn.yml to create a cloudformation stack, github token is: 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



## running app in EKS
## endpoints in ELB

$ nslookup a91feb16b26054ff29698b687dfbf376-1862089464.us-east-2.elb.amazonaws.com
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
Name:	a91feb16b26054ff29698b687dfbf376-1862089464.us-east-2.elb.amazonaws.com
Address: 3.134.191.227
Name:	a91feb16b26054ff29698b687dfbf376-1862089464.us-east-2.elb.amazonaws.com
Address: 18.221.186.36


$ curl --data '{"email":"abc@xyz.com", "password":"test"}' --header "Content-Type: application/json" -X POST a91feb16b26054ff29698b687dfbf376-1862089464.us-east-2.elb.amazonaws.com/auth
{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NTYxNzY5NzUsIm5iZiI6MTY1NDk2NzM3NSwiZW1haWwiOiJhYmNAeHl6LmNvbSJ9.9YEha75iDurxMvYuwLb5BBKpr9QE-JtzMzNcoJksoqo"}


$ TOKEN=`curl --data '{"email":"abc@xyz.com", "password":"test"}' --header "Content-Type: application/json" -X POST a91feb16b26054ff29698b687dfbf376-1862089464.us-east-2.elb.amazonaws.com/auth`
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   212  100   170  100    42   1827    451 --:--:-- --:--:-- --:--:--  2279


$ curl --request GET 'http://a91feb16b26054ff29698b687dfbf376-1862089464.us-east-2.elb.amazonaws.com/contents' -H "Authorization: Bearer ${TOKEN}"
{"email":"abc@xyz.com","exp":1656177740,"nbf":1654968140}
