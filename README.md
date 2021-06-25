

This AWS CloudFormation template can be used to automatically 
shutdown Amazon SageMaker Notebook instances when they are not in use. This is useful for cost savings.

With this template, when an Amazon SageMaker Notebook Instance is created or started, the following happens:
- an AWS Lambda function checks if the notebook instance has a config rule that checks for inactivity
- if the config rule is **not** applied, the instance is `stopped`, and the config rule is applied to the instance
- when the notebook instance is started at a future time, the config rule will run. If the notebook instance is not in use
  for more than one hour, the config rule will stop the notebook instance to save costs

The template creates the following resources:

1. **An Amazon SageMaker Lifecycle Config Rule** - this rule is applied to all new notebook instances, and performs
   a shutdown of the instance after one hour of inactivity
2. **CloudWatch Event Rule** - this event rule triggers when a SageMaker Notebook Instance is created or goes through any state-change.
   The event rule fires a lambda function.
3. **Lambda Function** - this function is triggered by the event rule. The function checks the Amazon SageMaker Notebook instance
to determine if it has the config rule applied. If the config rules is not applied, it shuts it down, and applies the config rule.
4. **Lambda Function Role** - with permissions to update notebook instances, stop notebook instances, and start notebook instances. 
5. **CloudWatch Permission** - for the event rule to invoke the lambda function.


Considerations when using this template:

1. The AWS CloudFormation template must be deployed to each region you want to apply the config rules to.
2. The SageMaker Notebook instances must be deployed to a VPC than can access the autostop script at: https://raw.githubusercontent.com/aws-samples/amazon-sagemaker-notebook-instance-lifecycle-config-samples/master/scripts/auto-stop-idle/autostop.py
3. Normally, when a user creates an Amazon SageMaker instance, the instance goes into an 'InService' state. 
 Using this automatic config rule, the notebook instance will be stopped so the config rule can be applied. The user must specifically start the instance after creating it.