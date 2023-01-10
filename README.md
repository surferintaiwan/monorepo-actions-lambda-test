# monorepo-github-actions-lambda-test
I want to put many lambda function into one repo and use github actions to deploy them to AWS.
This is a test repo to see if it works.

# Requirements
- aws sam cli


# Steps
## Step 1. Use sam init to create a new project

```
sam init

Which template source would you like to use?
        1 - AWS Quick Start Templates
        2 - Custom Template Location
Choice: 1
Which runtime would you like to use?
        1 - nodejs12.x
        2 - nodejs10.x
        3 - python3.8
        4 - python3.7
        5 - python3.6
        6 - python2.7
        7 - ruby2.7
        8 - ruby2.5
        9 - java8.al2
        10 - java8
        11 - go1.x
        12 - dotnetcore3.1
        13 - dotnetcore2.1
        14 - nodejs14.x
        15 - provided
Runtime: 1
Project name [sam-app]: monorepo-github-actions-lambda-test
```
And you will get a folder named `monorepo-github-actions-lambda-test`.
```
-- monorepo-github-actions-lambda-test js
    |-- .gitignore
    |-- README.md
    |-- hello-world
    |   |-- app.mjs
    |-- template.yaml
```

## Step 2. make sure hello-world function works
1. try `sam build` to build the container
2. try `sam local invoke "helloWorldFunction"` to test the function locally
3. try `sam lcal start-api` to test the api locally
4. try `sam deploy --guided` to deploy the function to AWS
5. if everything works, you can run `aws cloudformation delete-stack --stack-name monorepo-github-actions-lambda-test` to delete the stack
because we just want to test if the confiuguration can work, we don't need to keep the stack

## Step 3. copy the helloWorld folder to two new folders, lambda-a and lambda-b, and delete the helloWorld folder
1. remember to change the function name in template.yaml
2. lambda-a and lamba-b all have its template.yaml now

## Step 4.Add test.template.yaml into lambda-a and lambda-b folders
I want to deploy lambda-a and lambda-b to different stage, so I need to add a test.template.yaml to each folder.
1. template.yaml is used to deploy the function to dev stage
2. test.template is used to deploy the function to test stage
3. Actually, you can only use template.yaml to deploy dev stage and test stage, but I want to simulate that each lambda can have different deployment parameters in different stages, so I will add a test.template.yaml

## Step5. Add github actions workflow for lambda-a and lambda-b and test-lambda-a and test-lambda-b
I want to deploy lambda-a and lambda-b to dev stage and test stage, so I need to add four workflows
1. We hope that when different lambda folders have changes, the different workflow will be triggered, so we need to add the following to each workflow
    1. on: push
    2. paths: lambda-a/**, lambda-b/**
2. the main difference between these workflows is
    1. sam build --template
    2. sam deploy --stack-name
3. we use git hub actions secrets to store AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, these two are used to let github actions use aws cli to operate aws
AWS_ACCESS_KEY_ID 和 AWS_SECRET_ACCESS_KEY can be found in aws console's IAM -> Create Users
4. don't forget to change the name and jobs of workflow, it can help you to identify which workflow is for which lambda and which stage

## Step 6. Done
Now, you can try to modify lambda-a or lambda-b on development stage or test stage, and it will trigger the corresponding workflow to deploy the lambda to AWS.