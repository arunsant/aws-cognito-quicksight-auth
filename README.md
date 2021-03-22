---  
Important Note - This flow doesn't specify a RoleSessionName and hence the default value will be used to land user into QuickSight.  
Each users coming into QuickSight has to be uniquely identified for following reasons.
1) License compliance.
2) Avoid throttling / adverse impact to user experience.
3) Facilitates use of Row and Column level security.   
**Please ensure that user name/identifier is mapped as RoleSessionName while getting CognitoIdentityCredentials.**
---

# aws-cognito-quicksight-auth
A simple JavaScript frontend and SAM template to spin up a serverless backend, federating Cognito User Pools users to QuickSight.
<p align="center">
  <img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2017/10/03/quicksight-federated-1.jpg"  />
</p>


### Required Tools

* [aws cli](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

## Instructions

Get started by cloning the repository then editing some files described with more detail in steps 3-5:

1. Using the AWS CLI, create an S3 bucket in the same region where you want all resources to be deployed:

```
aws s3 mb s3://<bucket-name> --region <AWS Region>
```

2. Package the template with the following command and execute the resulting 'aws cloudformation deploy' output using the AWS CLI, referring to the S3 bucket created earlier:

```
aws cloudformation package --template-file quicksight.yaml --output-template-file quicksight-output.yaml --s3-bucket <S3 Bucket> 
```

The next command should be something similar to:

```
aws cloudformation deploy --template-file /Users/<full path>/quicksight-output.yaml --stack-name CognitoQuickSight --capabilities CAPABILITY_IAM
```
CloudFormation will automatically create and configure the following resources in your account:

*	CloudFront distribution
*	S3 static website
*	Cognito User Pools
*	Cognito Identity Pools
*	IAM Role for Authenticated Users
*	API Gateway API
*	Lambda Function

You can follow the progress of the stack creation from the AWS Console in CloudFormation. When the status of the “CognitoQuickSight” stack is CREATE_COMPLETE, execute the following command with the AWS CLI:

```
aws cloudformation describe-stacks --query 'Stacks[0].[Outputs[].[OutputKey,OutputValue]]|[]' --output text --stack-name CognitoQuickSight
```

3. Either refer to the output of the "describe-stacks" command above or go to the CloudFormation console, select the stack created on item 2 and open the OUTPUTS tab. All resources we'll need will be there. Use the information to fill up the details in the file "auth.js" including the region.

4. In the AWS Console, go to the Cognito User Pools section and select the pool named QuickSightUsers generated by CloudFormation. Under APP INTEGRATION -> DOMAIN NAME, create a Domain (be mindful domain names are unique to the region) and add the domain to the “auth.js” file accordingly.

5. Under APP INTEGRATION -> APP CLIENT SETTINGS select the option COGNITO USER POOL. Add the CloudFront distribution address (with https://, as SSL is a requirement for the callback/sign out URLs) and make sure that the address matches the related settings in the “auth.js” file exactly. For ALLOWED OAUTH FLOWS, select IMPLICIT GRANT. For ALLOWED OAUTH SCOPES, select OPENID.

6. Upload the four JS and HTML files from the root folder to the S3 bucket named “cognitoquicksight-s3website-xxxxxxxxx”. Make sure that all files are publicly readable.

7. Access the CloudFront distribution address from a browser to authenticate and access QuickSight
