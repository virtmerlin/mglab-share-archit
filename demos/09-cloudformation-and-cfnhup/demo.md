### 09-cloudformation-and-cfnhup
##### GIVEN:
  - An AWS Cloud9 desktop from 01-setup-cloud9
  - An simple LAMP app to deploy and version update with 2 CFN templates

##### WHEN:

  - I create a stack with V1 of the LAMP stack
  - I create a changeset for V2 of the LAMP stack & review it
  - I apply the changeset
  - I modify a security group by hand and detect drift fromt the stack

##### THEN:
  - I will be able to upgrade my LAMP app via versioned CFN templates
  - I will be able to detect drift of the actual resources from the desired state of the stack

##### SO THAT:
  - I can see basic CFN capabilities

##### NOTE: _This demo will provide CLI steps, but may have been demonstrated via the UI during a class delivery_

##### [Return to Main Readme](https://github.com/virtmerlin/mglab-share-archit#demos)

---------------------------------------------------------------
---------------------------------------------------------------
#### REQUIRES
- 01-setup-cloud9

---------------------------------------------------------------
---------------------------------------------------------------
#### DEMO

##### 1: Create CloudFormation 'Stack' that will create an IAM Role for the EC2 Instance, an Instance, and a security group to allow inbound traffic.
  - [DOC-LINK: CFN Hup LAMP Example](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-appframeworks-us-west-1.html)
- Reset your region & aws account variables in case you launched a new terminal session
```
cd ~/environment/mglab-share-archit/demos/09-cloudformation-and-cfnhup/
export C9_REGION=$(curl --silent http://169.254.169.254/latest/dynamic/instance-identity/document |  grep region | awk -F '"' '{print$4}')
echo $C9_REGION
export C9_AWS_ACCT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep accountId | awk -F '"' '{print$4}')
echo $C9_AWS_ACCT
```
- Create 'Stack'
```
aws cloudformation deploy --region $C9_REGION --template-file ./artifacts/cfn-hup-v1.json \
    --parameter-overrides KeyName=mg-awsacct-key-01-sfo \
    --stack-name archit-demos-09-cloudformation-and-cfnhup --tags CLASS=ARCHIT --capabilities CAPABILITY_NAMED_IAM
```

##### 2: Create a ChangeSet & Review it
- Optional Challenge:
  - Create & Apply a ChangeSet in the Console.  Use _./artifacts/cfn-hup-v2.json_ as the new template version

##### 3: Modify a security group manually & check drift
- Optional Challenge:
  - Change Security group rules via console
  - Initiate CloudFormation drift check via console ... What is the best way to fix ?

---------------------------------------------------------------
---------------------------------------------------------------
#### CLEANUP
Only run these scripts if you are done cleaning &/or running all dependent demos.
```
aws cloudformation delete-stack --region $C9_REGION  --stack-name archit-demos-09-cloudformation-and-cfnhup
```
