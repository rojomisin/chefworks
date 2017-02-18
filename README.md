## chefworks
spin up chef automate stack using `awscli opsworks-cm`

```
aws ec2 create-key-pair --key-name chefworks-00 --output text --query 'KeyMaterial'
aws cloudformation create-stack --stack-name chefworks-servicerole-00 --template-body file://./service-role-creation.yaml --capabilities CAPABILITY_IAM
aws cloudformation describe-stack-resources --stack-name chefworks-servicerole-00 --logical-resource-id InstanceProfile --region us-west-1
aws iam list-instance-profiles --region us-west-1 | jq '.InstanceProfiles[].Arn

```


```
"myStack" : {
  "Type" : "AWS::OpsWorks::Stack",
  "Properties" : {
    "Name" : {"Ref":"OpsWorksStackName"},
    "ServiceRoleArn" : { "Fn::Join": ["", ["arn:aws:iam::", {"Ref":"AWS::AccountId"}, ":role/aws-opsworks-service-role"]] },
    "DefaultInstanceProfileArn" : { "Fn::Join": ["", ["arn:aws:iam::", {"Ref":"AWS::AccountId"}, ":instance-profile/aws-opsworks-ec2-role"]] },
    "DefaultSshKeyName" : {"Ref":"KeyName"}
  }
}
```

## sample template from docs
```json
{
  "Type" : "AWS::OpsWorks::Stack",
  "Properties" : {
    "AgentVersion" : String,
    "Attributes" : { String:String, ... },
    "ChefConfiguration" : { ChefConfiguration },
    "CloneAppIds" : [ String, ... ],
    "ClonePermissions" : Boolean,
    "ConfigurationManager" : { StackConfigurationManager },
    "CustomCookbooksSource" : { Source },
    "CustomJson" : JSON,
    "DefaultAvailabilityZone" : String,
    "DefaultInstanceProfileArn" : String,
    "DefaultOs" : String,
    "DefaultRootDeviceType" : String,
    "DefaultSshKeyName" : String,
    "DefaultSubnetId" : String,
    "EcsClusterArn" : String,
    "ElasticIps" : [ ElasticIp, ... ],
    "HostnameTheme" : String,
    "Name" : String,
    "RdsDbInstances" : [ RdsDbInstance, ... ],
    "ServiceRoleArn" : String,
    "SourceStackId" : String,
    "UseCustomCookbooks" : Boolean,
    "UseOpsworksSecurityGroups" : Boolean,
    "VpcId" : String
  }
}
```
