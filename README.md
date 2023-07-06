# CI-CD-project-using-AWS-CodeCommit
This project develops a Continous Integration and Continous Delivery Pipeline by using only AWS services like CodeCommit, Elastic Beanstalk and RDS

## Overview

*Continuous integration(CI)* is a DevOps software development practice where developers regularly merge their code changes into a central repository, after which automated builds and tests are run. The key goals of continuous integration are to find and address bugs quicker, improve software quality, and reduce the time it takes to validate and release new software updates.

*Continous Delivery(CD)* is a software development practice where code changes are automatically prepared for a release to production. Continuous delivery expands upon continuous integration by deploying all code changes to a testing environment and/or a production environment after the build stage. When properly implemented, developers will always have a deployment-ready build artifact that has passed through a standardized test process.

This project focuses on implementing a full CI-CD pipeline using AWS services for the vprofile-project.
It serves as an alternative choice of developing and deploying a pipeline without using external tools like Jenkins and Github. Instead we use AWS CodeCommit to store the source code and AWS Codebuild service to build the code and pipeline service to execute the pipeline.

## Services Used
- AWS CodeCommit
- AWS CodeBuild
- Elastic Beanstalk
- Amazon Relational Database Service (RDS)
- Amazon Elasticache
- AmazonMQ
- S3
- EC2

## Project Architecture

The following things take place when a commit takes place:
- Whenever a commit is made the CodeCommit pipeline is triggered automatically.
- The code is prepared and sent to Codebuild service from the CodeCommit repository.
- The Codebuild service then builds the artifact from the source code and uploads the artifact into a S3 bucket. 
- Elastic Beanstalk (EB) fetches the artifact from the S3 bucket and hosts the application from the bucket.
- The EB  security group is connected with the backend security group to allow visiting traffic to the backend services.
- Backend service group includes services like:
    - AmazonMQ service
    - Amazon Relational Database Service (RDS)
    - Amazon Elasticache 
## Implementation Details
### Create Backend Services
- Create an EC2 key-pair to be used to launch ec2 instances.
- Create an IAM user to be used with Elastic Beanstalk with the following rules:
    - AWSElasticBeanstalkRoleSNS
    - AdministratorAccess-AWSElasticBeanstalk
    - AWSElasticBeanstalkCustomPlatformforEC2Role
    - AWSElasticBeanstalkWebTier
- Create RDS instance
    - Search for Amazon RDS on the console.
    - Go to RDS and create an instance.
    - Choose standard create and select MySQL as database engine.
   ![RDS_1](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/783e7acc-6e9a-45d8-bc2f-05955697c89d)

    - Choose the required version for the project (MySQL 5.7 is required for my project).
   ![RDS_2](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/9b6bcc54-81c4-4098-b054-ff103d3fc2a9)

    - Give your database a name and provide credentials.
    ![RDS_4](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/2e757163-11f5-4c10-82ef-2bfad25354ce)

    - Specify allocated storage.
   ![RDS_5](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/445469c8-17d3-46f4-880b-7f114ae28809)

    - under Additional configuration -> Database options provide the database name you want to create, otherwise no database will be created.
    ![RDS_6](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/5c263a12-fd5d-4254-9125-8cc160154233)

    - Store the database credentials by clicking on the view connection details button after instance is created.
   ![RDS_7](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/7221ce31-8da4-4d76-84a3-a8c80a983c80)

- Create and configure message broker (RabbbitMQ)
    - Search for AmazonMQ on the console.
    - Go to AmazonMQ and click on create a broker.
        - Select broker engine.
       ![rmq_1](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/74194933-7d78-4046-b297-2a314096fa6c)

        - Select deployment mode.
        ![rmq_2](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/5d1f122e-ed21-4ecd-ac96-cb0f3d959ef3)
        - Configure the broker settings. 
        ![rmq_3](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/1f6ef158-bef4-4840-b2b4-c7e9bf58e60b)
        - Review and create the broker.

- Configure and create cache service (Memcached Cluster)
    - Search for Elasticache on the console.
    - Go to Elasticache console.
    - Create a parameter group:
        - On the left-top hamburger menu select parameter groups.
        - Click on create parameter group.
        - Provide group name, description and family and create.
        ![ec-para-grp](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/e7677b4a-0b7b-4729-95e5-8a6bc26f31e9)
    - Create a subnet group:
        - On the left-top hamburger menu select subnet groups.
        - Click on create subnet group.
        - Provide group name, description and VPC then create.
        ![ec-sub-grp](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/3800f1eb-4aec-4fa7-b78e-439f62359680)
    - Create memcached cluster:
        - On the left-top hamburger menu select memcached cluster.
        - Click on create cluster.
        - Configure cluster settings.
        ![ec_memcache_1](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/079b9d19-3af7-46bd-beb2-2c8e0964331b)
        - provide memory as t2.micro to avail free tier services.
        ![ec_memcache_2](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/9040ea0a-cfca-46a3-a722-62004f51ac9e)
        - Choose the subnet group you created.
        ![ec_memcache_3](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/e99fae95-9f3a-465b-9444-2fa8c3b34ffd)
        - Advanced settings.
        ![ec_memcache_4](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/da3b1691-90a5-4d10-9d61-475a3233ef5b)

### Create Elastic Beanstalk Environment
- Search for Elastic Beanstalk on the console.
- Go to Elastic Beanstalk and click on Create application.
- Configure the environment:
    - Provide the application name.
![eb_1](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/764486e9-bdae-4195-ad2f-0265047a64ec)
    - Provide environment name and check and specify domain.
![eb_2](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/c0a365ab-cbe0-4a57-a971-acbe9ec3054b)
    - Select the application platform and its version.
![eb_3](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/2bde87f2-9e4c-4da2-ac2a-35daaf6db123)
    - Choose application code, presets and click on next.
![eb_4](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/63dd84e5-83ee-4599-a94f-22be2c785c37)

- Configure Service Access:
    - Create a new service role, provide the created ec2 key-pair and the created IAM user from earlier.
![eb_5](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/fbf47887-f6ec-41f3-8291-7be8ea32a846)
    - Check the activated option under instance settings.
![eb_6](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/f51bb501-8540-400a-854a-8d520b689f9c)

- Setup networking, database and tags:
    - Leave it as it is.

- Configure instance traffic and scaling:
    - Provide details for instance configuration.
![eb_7](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/18d11c6e-eefa-485e-9efd-29855547a3ea)
    - Specify necessary configuration for instance scaling.
![eb_8](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/ec356440-7138-4527-91fd-9ad4a8ecfd18)
    - Choose the instance type.
![eb_9](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/a270f844-cdc0-43d3-ba2c-7c13c4d0fb93)

- Configure updates, monitoring and rolling:
    - Select rolling under deployment policy of application deployments and specify the percentage or fixed number.
![eb_10](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/e78ab150-c5df-4cc5-9bd2-6c6941ebca44)
    
- Review and create application.
![eb_12](https://github.com/SumitM01/Hosting-vprofile-app-using-AWS-PAAS-and-SAAS/assets/65524561/e753447f-9a95-41c2-b701-1f3a6fd4e4d9)

### CONFIURE AWS CODECOMMIT AND COMMIT DATA FROM GITHUB TO CODECOMMIT:
- **CREATE A REPOSITORY** :
    - Give some name and description and create
	
- **CONFIGURE USER POLICIES:**
    - IAM -> create user, programmatic access -> create policy :
        - Service : codecommit, actions : manual actions : all, resources : specific : provide region and name of the repo -> review and create policy.
		- Select policy from existing policies -> create user -> delete access keys and secret keys -> upload ssh keys for codecommmit.
	- Generate keys in the local machine:
		- cd into C:Users/cr7su/.ssh folder then run 
        - name the key file and create using the folllowing command.
		```bash
        ssh-keygen
        ``` 
	- Upload the public key contents to the console -> upload ssh keys for codecommmit.
	- Generate a config file with the following contents:
		host : repo arn
		user : public rsa key
		identity : path to private rsa key
	- Edit permissions of the config file.
    ```bash
    chmod 600 config
    ```
    - Run the following command to authenticate over ssh.
    ```bash
    ssh arn_of_repository 
    ```
- **COMMIT DATA FROM GITHUB TO CODECOMMIT:**
    - Copy the config file, public-key and private-key file into the git-repository.
    - Execute the following commands to checkout all the branches from the github repository.
    ```bash
		git checkout master
        git branch -a | grep -v HEAD | cut -d ' /' -f3 | grep -v master > branches
    ```
    (in linux) 
    ```bash 
        for i in 'cat branches'; do git checkout $i; done
    ```
    (in windows powershell)
    ```bash
        $branches = Get-Content branches
        foreach ($branch in $branches) {
            git checkout $branch
       }
    ```
    - Fetch all tags 
    ```bash
        git fetch --tags
    ```
    - Remove remote repository.
    ```bash
        git remote rm origin
    ```
    - Add the CodeCommit repository as the new remote repository.
    ```bash
        git remote add origin "codecommit repo ssh url"
    ```
    - Push all the check-out branches from github repository into the Codecommit repository.
    ```bash
        git push origin --all
    ```
    - Push any tags present.
    ```bash
        git push --tags
    ```
### CREATE A BUILD JOB :
- **CODECOMMIT CODEBUILD**
    - Create project:
         - Select repository and branch as *vp-rem* 
         - Managed image go for ubuntu
         - Create a new service role
         - Select 3gb memory and 2 vCPUs 
         - insert build commands by opening editor and pasting contents of buildspec.yaml file
         - Create s3 bucket and select that bucket for artifact storage
         - Configure cloudwatch logs 
         - Edit the codebuild-vprofile-Build-service-role in IAM and add AmazonS3FullAccess permission to it.
    - Run the build

### CHANGE Elastic Beanstalk CONFIGURATION:
- **CONFIGURATION -> CONFIGURE INSTANCE TRAFFIC AND SCALING**
	- Go to processes and edit the existing process:
        - Under Healthcheck section change path to /login.
        - Under Session section check the stickiness enabled option.
    - Save and apply the changes then wait for the changes to be completed successfully.


### CREATE THE PIPELINE : 
- **CODECOMMIT PIPELINE**
    - Name the pipeline
    - Select source as Codecommit, select your repository and branch
    - Select Codebuild as your build provider and specify your existing project
    - Select Deployer as Elastic beanstalk, select your app, select your env
    - Review and create pipeline.

As soon as the pipeline is created, it is triggered and the execution takes place.

## Results
## Conclusion
As documented in this markdown format file, I have invested a significant amount of time in researching, learning, debugging to implement this project. If you appreciate this document please share with friends and do give it a try. 
## References
- https://www.udemy.com/course/decodingdevops
- www.google.com
- bingchat