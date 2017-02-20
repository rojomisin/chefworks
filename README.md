# chefworks
spin up chef automate stack using `awscli opsworks-cm` available in
- us-east-1
- us-west-2
- eu-west-1

### requirements
 - `awscli`
 - aws creds to do `aws configure --profile MyProfileWest`

### create a keypair
```
aws ec2 create-key-pair --key-name chefworks-vpc-00 --output text --query 'KeyMaterial' --profile SPSDevWest2 >chefworks-vpc-00.pem
aws ec2 create-key-pair --key-name chefworks-server-00 --output text --query 'KeyMaterial' --profile SPSDevWest2 >chefworks-server-00.pem
```

### spin up a base vpc in us-west-2
```
aws cloudformation create-stack --stack-name chefworks-vpc-00 \
    --template-body file://./base-vpc.template \
    --profile SPSDevWest2 \
    --parameters "ParameterKey=KeyName,ParameterValue=chefworks-vpc-00"

```

### wait for it...
```
aws cloudformation wait stack-create-complete --stack-name chefworks-vpc-00 --profile SPSDevWest2
```

### grab the subnets and vpc id
```
aws cloudformation describe-stacks --stack-name chefworks-vpc-00 --profile SPSDevWest2 \
    | jq '.Stacks[0].Outputs[] | .OutputKey,.OutputValue' | egrep 'outSN|outVPC' -A1
```


## notes
### aws command line stuff to create profiles (required by chef automate server in opsworks)
```
aws cloudformation create-stack --stack-name chefworks-servicerole-00 --template-body file://./service-role-creation.yaml --capabilities CAPABILITY_IAM
aws cloudformation describe-stack-resources --stack-name chefworks-servicerole-00 --logical-resource-id InstanceProfile --region us-west-2
aws iam list-instance-profiles --region us-west-2 | jq '.InstanceProfiles[].Arn

```

### sample stack
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

### ec2 classic
```
aws opsworks-cm create-server \
   --server-name chefworks-00 \
   --region us-west-2 \
   --instance-profile-arn arn:aws:iam::086761730243:instance-profile/chefworks-servicerole-00-InstanceProfile-ZE4L9UBFCX1S \
   --service-role-arn arn:aws:iam::086761730243:role/chefworks-servicerole-00-ServiceRole-1OHOD48O1QNRU \
   --profile SPSDevWest2 \
   --instance-type m3.large \
   --engine Chef \
   --engine-version 12 \
   --engine-model Single \
   --subnet-ids "subnet-baab7af3" \
   --key-pair chefworks-server-00 >chefworks-server-00_output.json

```

## sample template from docs
```
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
