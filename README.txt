Login to aws account using browser on your laptop:
Once logged in to the aws console:  


# Region selection
select north virginia region


# Group creation
select IAM as service:
create IAM group: demogroup
select the managed policy to be attached to this group:
s3 full access: AmazonS3FullAccess
api gateway full admin access: AmazonAPIGatewayAdministrator
lambda full access: AWSLambdaFullAccess
cloudformation full access: AWSCloudFormationFullAccess


# User creation
select IAM as service:
create IAM user with programmatic access only: demouser
add user to group: demogroup
FYI the below is access and secret is for example ONLY
make a note of access key id: AKIATNG2U5D66OSF7NUP
make a note of secret access key: YT9C2IMiSITdZCupuDbPJo712T7md7MFdF06IJHBC


# Initial Source Code
on your laptop:
create directory: demo




# samtemplate.yaml will be the sam template for creating a lambda function which will be triggered (event) by an API Gateway with resource as hello and method as GET
create file: samtemplate.yaml
		
AWSTemplateFormatVersion: 2010-09-09
Description: This will create a lambda function and an api event which will trigger it. Also, lambda execution role

Transform: 'AWS::Serverless-2016-10-31'
Resources:
  demofunction:
	Type: AWS::Serverless::Function
	Properties:
	  Description: This is just a hello world lambda with api integration
	  Handler: lambdademofunction.lambda_handler
	  Runtime: python3.7
	  CodeUri: './lambdademofunction.py'
	  Events:
		demoapi:
			# Define an API Gateway endpoint that responds to HTTP GET at /hello
			Type: Api
			Properties:
			  Path: /hello
			  Method: GET
	  MemorySize: 128
	  Timeout: 15
	  Policies:
	  Tags:
		Name: demo




# buildspec.yaml will be used by AWS Code Build Service
create file: buildspec.yaml
		
version: 0.2
phases:
  install:
    runtime-versions:
      python: 3.7 
    commands:
      - echo "Installing phase"
  build:
    commands:
      - echo "Starting SAM packaging `date` in `pwd`"
      - aws cloudformation package --template-file samtemplate.yaml --s3-bucket demolalit --output-template-file packagedcloudformationtemplate.yaml
      - aws cloudformation deploy --template-file packagedcloudformationtemplate.yaml --stack-name demo --capabilities CAPABILITY_IAM




# lambdademofunction.py is the python3.7 source code
create file: lambdademofunction.py

import json

def lambda_handler(event, context):
	# TODO implement
	return {
		'statusCode': 200,
		'body': json.dumps('Hello from Lalit')
	}




# Source Code repository
using browser on your laptop login to github.com: https://github.com/Lalitduggal
create public git repo: demo
upload above files
commit



# prepare the system to be used for build and packaging
on your laptop:
install aws cli



# go to command prompt or terminal and change directory to the place where the code is present
eg: 
cd Desktop/myobdemo/demo/

aws configure

IAM user: demouser
access: AKIATNG2U5D66OS45NUP
secret: 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
default region: us-east-1
default output format: json




# make S3 bucket
aws s3 mb s3://demolalit




# package the serverless code using sam template and store the converted cloudformation template in s3 bucket:	
aws cloudformation package --template-file samtemplate.yaml --s3-bucket demolalit --output-template-file packagedcloudformationtemplate.yaml



# deploy the serverless code using cloudformation template
aws cloudformation deploy --template-file C:\Users\lalitduggal\Desktop\myobdemo\demo\packagedcloudformationtemplate.yaml --stack-name demo --capabilities CAPABILITY_IAM


# test the api
From your laptop browser:  https://rjviq5w9g9.execute-api.us-east-1.amazonaws.com/Prod/hello



# once the above steps are achieved. Now, lets plan for CICD



# code build
login to aws console:
select AWS Code Build Service
create a code build project: demobuild
a role is also created by default: arn:aws:iam::2322667867133:role/service-role/codebuild-demobuild-service-role
grant below policies to this role using IAM AWS Console: 
s3 full access: AmazonS3FullAccess
api gateway full admin access: AmazonAPIGatewayAdministrator
lambda full access: AWSLambdaFullAccess
cloudformation full access: AWSCloudFormationFullAccess



# code pipeline
login to aws console
select AWS Code Pipeline Service
codepipeline: demopipe
role is auto created: AWSCodePipelineServiceRole-us-east-1-demopipe: arn:aws:iam::2322667867133:role/service-role/AWSCodePipelineServiceRole-us-east-1-demopipe
select source code provider: github
connect to github
repository: Lalitduggal/demo
branch: master
select gitwebhooks
build provider: codebuild
region: us-east-1
project name of codebuild: demobuild
deploy provider: skip




# perform a test to check whether the code pipeline gets triggered as soon as you change the lambda function code in git repo and commit.



# codepipeline is getting triggered because of the commit to git repo: demo 
# check api result after 2 minutes
API URL: https://rjviq5w9g9.execute-api.us-east-1.amazonaws.com/Prod/hello



######################################################################################
# For checking the working of my CICD by reviewer
######################################################################################
from the laptop using browser: https://github.com/Lalitduggal/demo/blob/master/lambdademofunction.py
edit this file in browser.
Just change the text: "Hello again from Lalit Duggal" to anything of your choice eg: "This is test cicd"
commit
This will trigger AWS Code Pipeline which will build and deploy the new API
check the api in 2 minutes: It will show the next text you had committed.
API URL: https://rjviq5w9g9.execute-api.us-east-1.amazonaws.com/Prod/hello
