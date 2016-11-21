# Creating a Lambda with Scala :see_no_evil:

An almost step-by-step guide on how to create an AWS lambda using Scala based on my experience
writing the [Apple News Analytics Lambda](https://github.com/guardian/apple-news-analytics-lambda); you can use
that repo as a reference.

### Lambda uses cases

You might want to consider using a lambda if:

- Your app responds to (AWS) _events_ rather than to user actions: you want some actions to be triggered by an S3 event, a DyanmoDB write, you want to connect it to a Kinesis stream and so on.
- You want to move your app from a poll logic to a push logic and have AWS handle it for you.
- You want your app to do something on scheduled events.

Check out more use cases [here](http://docs.aws.amazon.com/lambda/latest/dg/use-cases.html). Other goodies include:

- You can test your lambda from the AWS web console by writing your test event and run it again the lambda.
- Logs end up in Cloudwatch automatically.
- Fast feedback cycle - just reupload your jar, no actual machine redeploy needed.
- There's an official Java SDK that covers pretty much everything you could possibly need.

### Lambda limitations:

- Max timeout hardcoded to 5 minutes (deafult being 1 minute). It means that whatever your lambda does, it has to be done within 5 minutes top.
- Unpredictable spin up overhead when running software on the JVM (ScalaJS anyone?).
- No control whatsoever on the hardware (obviously). You can find a list of physical limitations [here](http://docs.aws.amazon.com/lambda/latest/dg/limits.html).

### Writing a Lambda

The only required step is to specify your request handler, which can be called however you want (the name can be set from the AWS web console). I like to keep the default name (`handleRequest`) and extend the [RequestHandler interface](https://github.com/aws/aws-lambda-java-libs/blob/master/aws-lambda-java-core/src/main/java/com/amazonaws/services/lambda/runtime/RequestHandler.java) so that it's obvious from the source code where the logic starts, e.g.

```scala
class Lambda extends RequestHandler[S3Event, Unit] {
    override def handleRequest(event: S3Event, context: Context): Unit = {
        // hic sunt leones 
    }
}
```

In this example I'm using as `S3Event` as event type (courtesy of the AWS SDK), but an event is really just a json blob you could handle yourself as a Java `Map[String, Object`] if you like. You're done! :) - however, if you want an even more verbose guide with pictures [here](https://aws.amazon.com/blogs/compute/writing-aws-lambda-functions-in-scala/)'s a good one.

### Deploying a Lambda

1. First you need to package up your project. One easy way to do it is with [sbt assembly](https://github.com/sbt/sbt-assembly).

    - Add sbt assembly to your `plugins.sbt`
    - Add the line `assemblySettings` to your `build.sbt`
    - Run `sbt reload` to make the changes effective.
    - You should now be able to run `sbt assembly` to produce your artifact.

2. Upload your artifact to S3

    - The first time you need to manually upload your artifact to S3. For CAPI, the path would be like
    `content-api-dist/content-api/$STAGE/$PROJECT/$ARTIFACT.zip`. Adjust the path
    to follow your team conventions.
    
3. Create a new CloudFormation stack

    - Write your `cloudformation.yaml` to create a new stack for your lambda. You can use
    [this one](https://github.com/guardian/apple-news-analytics-lambda/blob/master/send-events/cloudformation.yaml) as a blueprint.
    Make sure the `S3Key` is pointing to the artifact you previously uploaded.
    - Once your `cloudformation.yaml` is in place, go ahead and use it to create a new stack from the AWS CloudFormation
    web console. This will spin up a new lambda that uses the artifact you uploaded in point (1).
   
### Continuous integration

1. You need to write a `deploy.json` file to be consumed by RiffRaff; you can use
[this one](https://github.com/guardian/apple-news-analytics-lambda/blob/master/send-events/deploy.json) as a blueprint.
You'll have to replace the name of your lambda and the paths to point to what you specified in the `cloudformation.yaml`.

2. Create a new GitHub repository and push the code so far.

3. Log in into Team City and create a new project in the usual way (i.e. copy the config from some other project
that looks reasonably similar to yours; it doesn't have to be a lambda).

4. Use Team City to manually run your build. If the build is successful, you'll be able to deploy your project
as usual using RiffRaff.



