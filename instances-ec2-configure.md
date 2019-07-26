# Configure an Amazon EC2 Instance to Work with CodeDeploy<a name="instances-ec2-configure"></a>

These instructions show you how to configure an Amazon EC2 instance running Amazon Linux, Ubuntu Server, Red Hat Enterprise Linux \(RHEL\), or Windows Server for use in CodeDeploy deployments\.

**Note**  
If you do not have an Amazon EC2 instance, you can use the AWS CloudFormation template to launch one running Amazon Linux or Windows Server\. We do not provide a template for Ubuntu Server or RHEL\.

## Step 1: Verify an IAM Instance Profile Is Attached to Your Amazon EC2 Instance<a name="instances-ec2-configure-1-verify-instance-profile-attached"></a>

1. Sign in to the AWS Management Console and open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Instances**, choose **Instances**\.

1. Browse to and choose your Amazon EC2 instance in the list\.

1. In the details pane, on the **Description** tab, note the value in the **IAM role** field, and then proceed to the next section\.

   If the field is empty, you can attach an IAM instance profile to the instance\. For information, see [Attaching an IAM Role to an Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#attach-iam-role)\.

## Step 2: Verify the Attached IAM Instance Profile Has the Correct Access Permissions<a name="instances-ec2-configure-2-verify-instance-profile-permissions"></a>

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**\.

1. Browse to and choose the IAM role name you noted in step 4 of the previous section\.
**Note**  
If you want to use the service role generated by the AWS CloudFormation template instead of one you created by following the instructions in [Step 3: Create a Service Role for CodeDeploy](getting-started-create-service-role.md), note the following:  
In some versions of our AWS CloudFormation template, the display name of the IAM instance profile generated and attached to the Amazon EC2 instances is not the same as the display name in the IAM console\. For example, the IAM instance profile might have a display name of `CodeDeploySampleStack-expnyi6-InstanceRoleInstanceProfile-IK8J8A9123EX`, while the IAM instance profile in the IAM console might have a display name of `CodeDeploySampleStack-expnyi6-InstanceRole-C5P33V1L64EX`\.  
To help you identify the instance profile in the IAM console, you'll see the prefix of `CodeDeploySampleStack-expnyi6-InstanceRole` is the same for both\. For information about why these display names might be different, see [Instance Profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/instance-profiles.html)\.

1. Choose the **Trust Relationships** tab\. If there is no entry in **Trusted Entities** that reads **The identity provider\(s\) ec2\.amazonaws\.com**, you cannot use this Amazon EC2 instance\. Stop and create an Amazon EC2 instance using the information in [Working with Instances for CodeDeploy](instances.md)\.

   If there is an entry that reads **The identity provider\(s\) ec2\.amazonaws\.com**, and you are storing your applications in GitHub repositories only, then skip ahead to [Step 3: Tag the Amazon EC2 Instance](#instances-ec2-configure-3-tag-instance)\.

   If there is an entry that reads **The identity provider\(s\) ec2\.amazonaws\.com**, and you are storing your applications in Amazon S3 buckets, choose the **Permissions** tab\.

1. If there is a policy in the **Permissions policies** area, expand the policy, then choose **Edit policy**\.

1. Choose the **JSON** tab\. If you are storing your applications in Amazon S3 buckets, make sure `"s3:Get*"` and `"s3:List*"` are in the list of specified actions\. 

   It may look something like this:

   ```
   {"Statement":[{"Resource":"*","Action":[
     ... Some actions may already be listed here ...
     "s3:Get*","s3:List*"
     ... Some more actions may already be listed here ...
     ],"Effect":"Allow"}]}
   ```

   Or it may look something like this:

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Action": [
             ... Some actions may already be listed here ...
             "s3:Get*",
             "s3:List*"
             ... Some more actions may already be listed here ...
           ],
           ...
         }
       ]
     }
   ```

   If `"s3:Get*"` and `"s3:List*"` are not in the list of specified actions, choose **Edit** to add them, and then choose **Save**\. \(If neither `"s3:Get*"` or `"s3:List*"` is the last action in the list, be sure to add a comma after the action, so the policy document validates\.\)
**Note**  
We recommend that you restrict this policy to only those Amazon S3 buckets your Amazon EC2 instances must access\. Make sure to give access to the Amazon S3 buckets that contain the CodeDeploy agent\. Otherwise, an error might occur when the CodeDeploy agent is installed or updated on the instances\. To grant the IAM instance profile access to only some CodeDeploy resource kit buckets in Amazon S3, use the following policy, but remove the lines for buckets you want to prevent access to:  

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "s3:Get*",
           "s3:List*"
         ],
         "Resource": [
           "arn:aws:s3:::replace-with-your-s3-bucket-name/*",
           "arn:aws:s3:::aws-codedeploy-us-east-2/*",
           "arn:aws:s3:::aws-codedeploy-us-east-1/*",
           "arn:aws:s3:::aws-codedeploy-us-west-1/*",
           "arn:aws:s3:::aws-codedeploy-us-west-2/*",
           "arn:aws:s3:::aws-codedeploy-ca-central-1/*",
           "arn:aws:s3:::aws-codedeploy-eu-west-1/*",
           "arn:aws:s3:::aws-codedeploy-eu-west-2/*",
           "arn:aws:s3:::aws-codedeploy-eu-west-3/*",
           "arn:aws:s3:::aws-codedeploy-eu-central-1/*",
           "arn:aws:s3:::aws-codedeploy-ap-east-1/*",
           "arn:aws:s3:::aws-codedeploy-ap-northeast-1/*",
           "arn:aws:s3:::aws-codedeploy-ap-northeast-2/*",
           "arn:aws:s3:::aws-codedeploy-ap-southeast-1/*",        
           "arn:aws:s3:::aws-codedeploy-ap-southeast-2/*",
           "arn:aws:s3:::aws-codedeploy-ap-south-1/*",
           "arn:aws:s3:::aws-codedeploy-sa-east-1/*"
         ]
       }
     ]
   }
   ```

## Step 3: Tag the Amazon EC2 Instance<a name="instances-ec2-configure-3-tag-instance"></a>

For instructions about how to tag the Amazon EC2 instance so that CodeDeploy can find it during a deployment, see [Working with Tags in the Console](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html#Using_Tags_Console), and then return to this page\.

**Note**  
You can tag the Amazon EC2 instance with any key and value you like\. Just make sure to specify this key and value when you deploy to it\.

## Step 4: Install the AWS CodeDeploy Agent on the Amazon EC2 Instance<a name="instances-ec2-configure-4-install-agent"></a>

For instructions about how to install the CodeDeploy agent on the Amazon EC2 instance and verify it is running, see [Managing CodeDeploy Agent Operations](codedeploy-agent-operations.md), and then proceed to [Create an Application with CodeDeploy](applications-create.md)\.