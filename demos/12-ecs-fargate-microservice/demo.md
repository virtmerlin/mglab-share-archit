### 12-ecs-fargate-microservice
##### GIVEN:
  - An AWS Cloud9 desktop from 01-setup-cloud9
  - An a simple java app

##### WHEN:

  - I create an ECR Repo
  - I create an ECS Cluster
  - I create an ECS Task (fargate)
  - I Create an ALB
  - I Create an ECS Service

##### THEN:
  - I will get my TV app running on serverless fargate as a container

##### SO THAT:
  - I can see how to run a microservice -- serverless

##### NOTE: _This demo is an 'Advanced Demo' and assumes you will be able to navigate the AWS Console & ECS to create a Fargate Task & Service_

##### [Return to Main Readme](https://github.com/virtmerlin/mglab-share-archit#demos)

---------------------------------------------------------------
---------------------------------------------------------------
#### REQUIRES
- 01-setup-cloud9

---------------------------------------------------------------
---------------------------------------------------------------
#### DEMO

##### 1: Create an ECR Registry to store the Docker Image we are going to build :)
- Reset your region & aws account variables in case you launched a new terminal session
```
cd ~/environment/mglab-share-archit/demos/12-ecs-fargate-microservice/
export C9_REGION=$(curl --silent http://169.254.169.254/latest/dynamic/instance-identity/document |  grep region | awk -F '"' '{print$4}')
echo $C9_REGION
export C9_AWS_ACCT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep accountId | awk -F '"' '{print$4}')
echo $C9_AWS_ACCT
```
- Create ECR Registry
```
aws ecr describe-repositories --repository-names archit-demo-12-tv --region $C9_REGION || aws ecr create-repository --repository-name archit-demo-12-tv --region $C9_REGION
```

##### 1: Create an OCI (Docker) image & push it up to the ECR repo
- fetch the java app 'JAR'
```
mkdir app && wget https://mglab-aws-samples.s3.amazonaws.com/classes/archit/03/java-tv-spring-0.0.1.jar -O ./app/java-tv-spring-0.0.1.jar
```
- Use docker cli to build the OCI Image
```
docker build -f ./artifacts/Dockerfile . -t archit-demo-12-tv:latest -t archit-demo-12-tv:v1.0
```
- Test it locally on Cloud9 IDE
```
docker run --name crazy-tv -p 9000:8080 -d archit-demo-12-tv:latest
curl http://localhost:9000
docker stop crazy-tv
```
- Tag the built image to push it up to ECR, then push it, after running these commands you should see an image in the ECR repository
```
aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin $C9_AWS_ACCT.dkr.ecr.us-west-1.amazonaws.com
docker tag archit-demo-12-tv:latest $C9_AWS_ACCT.dkr.ecr.$C9_REGION.amazonaws.com/archit-demo-12-tv:latest
docker push $C9_AWS_ACCT.dkr.ecr.$C9_REGION.amazonaws.com/archit-demo-12-tv:latest
```

##### 3: Create a Security Group Allowing 80 & 8080
- Use the Console

##### 4: Create a ELB in public subnets use Security Group from Above
- Use the Console

##### 5: Create ECS Cluster
- Use the Console

##### 6: Create ECS 'Fargate' Task, lets use a public image so we don't have to deal with IAM access to ECR from Fargate Exec task
- Use the Console
  - ---> 1GB & .5 vCPU
  - ---> public.ecr.aws/u3e9a9s8/archit-demo-12-tv

##### 7: Create ECS Service to launch 5 Fargate Tasks into our ALB
- Use the Console

---------------------------------------------------------------
---------------------------------------------------------------
#### CLEANUP
Only run these scripts if you are done cleaning &/or running all dependent demos.
```
aws ecr delete-repository --repository-name archit-demo-12-tv --region $C9_REGION --force
```
- Use Console
  - --->ECS Cluster/Service/Tasks & ALB
