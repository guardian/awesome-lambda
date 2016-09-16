# Creating a Lambda with Scala <3

This is an almost step-by-step guide on how to run an AWS lambda using Scala. It is based on my experience
writing the [Apple News Analytics Lambda](https://github.com/guardian/apple-news-analytics-lambda); you can use
that repo as a reference.

### Create a new Lambda

1. First you need to package up your project. One easy way to do it is with [sbt assembly](https://github.com/sbt/sbt-assembly).

    - Add sbt assembly to your `plugins.sbt`
    - Add the line `assemblySettings` to your `build.sbt`
    - Run `sbt reload` to make the changes effective.
    - You should now be able to run `sbt assembly` to produce your artifact.

2. Upload your artifact to S3

    - The first time you need to manually upload your artifact to S3. For CAPI, the path would be like
    `content-api-dist/content-api/$STAGE/name-of-your-project/name-of-your-artifact.zip`. Adjust the path
    to follow your team conventions.
    
3. Create a new CloudFormation stack

    - Write your `cloudformation.json` to create a new stack for your lambda. You can use
    [this one](https://github.com/guardian/apple-newsi-analytics-lambda/blob/master/cloudformation.json) as a blueprint.
    Make sure the `S3Key` is pointing to the artifact you previously uploaded.
    - Once your `cloudformation.json` is in place, go ahead and use it to create a new stack from the AWS CloudFormation
    web console. This will spin up a new lambda that uses the artifact you uploaded in point (1).
   
### Continuous integration

1. You need to write a `deploy.json` file to be consumed by RiffRaff; you can use
[this one](https://github.com/guardian/apple-news-analytics-lambda/blob/master/deploy.json) as a blueprint.
You'll have to replace the name of your lambda and the paths to point to what you specified in the `cloudformation.json`.

2. Create a new GitHub repository and push the code so far.

3. Log in into Team City and create a new project in the usual way (i.e. copy the config from some other project
that looks reasonably similar to yours; it doesn't have to be a lambda).

4. Use Team City to manually run your build. If the build is successful, you'll be able to deploy your project
as usual using RiffRaff.
 
### Permissions

If your lambda does things like writing to Dynamo, uploading stuff to S3 and so on you'll need to grant it
the right permissions. You can do so from the AWS console > IAM > Roles and finding the role used by your lambda.



