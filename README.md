This is a meta pipeline for setting up AWS CodeCommit, CodePipeline, CodeBuild, & CodeDeploy

# Create a bucket to store the seeding artifacts for the setup; These are the src.zip in step 2 and the 
# codestar_toolchain.yml file in step3 
1. Create an S3 bucket, for the example below this is 'codestar-github-test'.

# Then create a zip of your repo; For this example, you can use: https://github.com/ihr/test-codestar
2. zip src.zip .gitignore README.md app/* app-sam.yaml buildspec.yml index.js test/* test-sam.yaml tests/*

# Copy the zip you create in the previous step
3. aws s3 cp src.zip s3://codestar-github-test/src.zip

# Copy the codestar_toolchain.yml
4. aws s3 cp ./codestar_toolchain.yml s3://codestar-github-test/codestar_toolchain.yml

# Modify the codestar_input.json so that it contains the right parameters; You will needed a GitHub Access Token which 
# you can create by following this tutorial: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/
# The access token has to have the following scopes: repo, user, and admin:repo_hook.
5. aws codestar create-project --cli-input-json file://codestar_input.json

# In case you need to modify the codestar_toolchain.yml and apply the changes, then you will have to create a change set:

aws cloudformation create-change-set --stack-name awscodestar-codestar-github \
--change-set-name AChangeSet --template-url https://s3.eu-central-1.amazonaws.com/codestar-github-test/codestar_toolchain.yml --parameters \
    ParameterKey=ProjectId,UsePreviousValue=true \
    ParameterKey=GitHubOwner,UsePreviousValue=true \
    ParameterKey=RepositoryName,UsePreviousValue=true \
    ParameterKey=GitHubSecret,UsePreviousValue=true \
    ParameterKey=GitHubOAuthToken,UsePreviousValue=true \
--capabilities CAPABILITY_NAMED_IAM

# ... and execute the change set:
aws cloudformation execute-change-set --change-set-name <arn:>    

Further resources:
- https://aws.amazon.com/blogs/compute/continuous-deployment-for-serverless-applications/ 
- https://docs.aws.amazon.com/codestar/latest/userguide/how-to-create-project.html#how-to-create-project-cli 