# jenkins-pipeline-deployment
this repo will serve as the master for a jenkins pipeline previously built for a code deployment. I will be documenting it so that it can be used by others

![jenkins-pipeline-code-deployment-diagram](images/jenkins-pipeline-code-deployment.png)


## prerequisites

- awscli configured for account you want to deploy resources to.
- docker installation and knowledge of how to use docker
- knowledge of jenkins

## build jenkins server

- AWS documentation for this build can be found in whitepapers so I won't be going into the details. See of commands below to build a baseline.
  - https://d1.awsstatic.com/Projects/P5505030/aws-project_Jenkins-build-server.pdf
  - https://d0.awsstatic.com/whitepapers/DevOps/Jenkins_on_AWS.pdf

- Build a amz linux
- ssh into your instance
```
ssh -i ~/.ssh/<your_key.pem> ec2-user@ec2<ip_of_ec2>.compute-1.amazonaws.com
```

- If your distro of linux does not have wget...you can download and install it to use commands below.
```
sudo yum install wget
```

- Now we can install and configure Jenkins
```
sudo yum update –y
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkinsci.org/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum install jenkins -y
sudo service jenkins start
```
- Head to the directory below and grab the admin password as you will need it in the next step.
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Now you connect to the Jenkins interface via the servers public dns over port 8080 and use the admin password to connect.
```
http://<your_server_public_DNS>:8080
```

- That's it! Now that your Jenkins instance is up and running. We'll go setup a certificate and Route53 alias for a cleaner/secure url. This step is optional if you don't have a domain registar or don't require SSL.

## setup up docker image and ecr (setup assumes you have awscli to aws account)

- log into ecr to create and store docker image.
```
aws ecr get-login
aws ecr create-repository --repository-name <your_repository_name>
aws ecr describe-repositories
aws ecr list-images --repository-name <your_repository_name>
```

- we are going to use an nginx image as an example here, but you can pull whichever image needed for your project
```
docker pull nginx:1.9
docker tag nginx:1.9 <your_aws_account_number>.dkr.ecr.us-east-1.amazonaws.com/ecs-test/nginx:1.9
docker push <your_aws_account_number>.dkr.ecr.us-east-1.amazonaws.com/ecs-test/nginx
aws ecr list-images --repository-name <your_repository_name>
```

## task definitions

## optional jenkins-codebuild via cloudformation template

- Optionally, a jenkins/codebuild can be created and configured more easily via a cloudformation template.
- Reference: https://aws.amazon.com/blogs/devops/simplify-your-jenkins-builds-with-aws-codebuild/
