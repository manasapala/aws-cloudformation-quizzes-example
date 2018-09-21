# HQ CloudFormation Templates

This repo contains the AWS CloudFormation templates and supporting files that can be used to the build the backend stack for the HQ Trivia app (Fall 2018) within Amazon Web Services. 

## Usage

The file database.yaml is designed primarily to provision the VPN and subnets, and position a new RDS instance within it to function as the SQL database for the system. It should only be run once, at the beginning of the project, to the create this part of the stack. Running this template will prompt the user for specific parameters, including the master password for the database. These parameters, as well as the ids of the generated private subnets and security groups, and the host of the database instance, will be used in the second phase of the set-up, and should be recorded.

The file pipeline.yaml is designed to run second, and should also be run only once. It uses some of the parameters and output values of database.yaml. It is designed to create a new AWS CodePipeline instance, and within this pipeline, a new AWS CodeBuild instance that builds and deploys the AWS Lambda functions and affiliated API Gateway that allow HTTPS interaction with the SQL database.

For its source, the CodeBuild project pulls from a branch of a second Github repo (i.e. not this one) and runs a build script (the buildspec.yml of the project) that is embedded within pipeline.yaml itself. The pipeline is configured to receive a webhook notification from the Github repo such that whenever the master branch is updated, CodePipeline will pull the updated repository files (all of them) from the tip of the master branch and initiate a new build process to either create or update the Lambda functions and the API Gateway.

The result of the build process is a packaged zip file of Python code (for the Lambda functions) that is bundled in a zip file and placed into S3 bucket ready for deployment. 

In the deployment stages of the pipeline, a new CloudFormation (sub-)stack is created in which a Change Set is created from the bundled Lambda functions, and then this Change Set is executed across the deployment fleet of instances on which the Lambda functions are or will be running. It also updates the API Gateway if necessary (i.e. with new or updated HTTP methods and paths).


