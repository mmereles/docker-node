# Build a docker image with AWS CodeBuild & GitHub

## preview steps:

- Create a AWS ECR repository
- Create a GitHub Repository and add the Dockerfile and the files that will use the image
- Generate a GitHub Key for clone the repo

![image](https://user-images.githubusercontent.com/60193314/198838928-f638a4cc-4f74-4366-8ac9-1c00b6c03b36.png)


## Get Started with AWS CodeBuild

the most importants steps for build a docker image in Codebuild are the Role that work as a account, the GitHub repositoy with the key, set as Priviledge the OS that build the image and the environment variables

- Source Section (Github)
  - add the key and then add the Github repository
- Generate the role with a custom name
- Add the environment variables, for this demo i put as plaintext you can use AWS Secret Manager
  - `AWS_DEFAULT_REGION` = your AWS region
  - `AWS_ACCOUNT_ID` = Account ID --> you can extract this from your AWS Profile
  - `IMAGE_REPO_NAME` = Name of your ECR repo
  - `IMAGE_TAG` = ID tag or latest

- set as Priviledge your OS Environment for build the Docker Image

<img width="548" alt="image" src="https://user-images.githubusercontent.com/60193314/198839450-eab83f1c-616a-4621-95ff-aac4b93a580d.png">

  
  then you need add your BuildSpec script, you can upload or you can paste in the same screen
  
```yml
  version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...          
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG      
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
      
```

Save your CodeBuild Project, but dont run yet because you need edit your policy role adding the ECR permissions for connect to ECR and Add the image to your repo

- Open another tab with your IAM Roles and edit the policy for the new CodeBuild Role, add the following script at the end


```json
{
  "Statement": [
    ### BEGIN ADDING STATEMENT HERE ###
    {
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:GetAuthorizationToken",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "*",
      "Effect": "Allow"
    },
    ### END ADDING STATEMENT HERE ###
    ...
  ],
  "Version": "2012-10-17"
}
```
Perfect!.... once fill the policy, save the changes and run your Code Build Project

Your finally can see the Docker image in your AWS ECR

<img width="715" alt="image" src="https://user-images.githubusercontent.com/60193314/198839625-8d4030b1-52ee-4cd7-8fdc-5c31d1e4ee52.png">




