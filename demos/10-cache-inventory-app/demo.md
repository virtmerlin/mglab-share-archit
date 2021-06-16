### 10-cache-inventory-app
##### GIVEN:
  - An AWS Cloud9 desktop from 01-setup-cloud9
  - An a sample PHP Inventory App using Memcached
  - The Mysql DB from the 04-rds-mysql-access Demo

##### WHEN:

  - I create a CFN stack with V1.1 Inventory App

##### THEN:
  - I will be get the Inventory App
  - I will also get an ASG + a Memcached Cluster

##### SO THAT:
  - I can see the impact & complexity of Caches in front of a RDBMS

##### NOTE: _This demo will provide CLI steps, but may have been demonstrated via the UI during a class delivery_

##### [Return to Main Readme](https://github.com/virtmerlin/mglab-share-archit#demos)

---------------------------------------------------------------
---------------------------------------------------------------
#### REQUIRES
- 01-setup-cloud9
- 04-rds-mysql-access

---------------------------------------------------------------
---------------------------------------------------------------
#### DEMO

##### 1: Create CloudFormation 'Stack' that will all reqd resources for V1.1 Inventory App with Memcached support.
- Reset your region & aws account variables in case you launched a new terminal session
```
cd ~/environment/mglab-share-archit/demos/10-cache-inventory-app/
export C9_REGION=$(curl --silent http://169.254.169.254/latest/dynamic/instance-identity/document |  grep region | awk -F '"' '{print$4}')
echo $C9_REGION
export C9_AWS_ACCT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep accountId | awk -F '"' '{print$4}')
echo $C9_AWS_ACCT
```
- Create 'Stack'
```
aws cloudformation deploy --region $C9_REGION --template-file ./artifacts/inventory-memcached.yaml \
    --stack-name archit-demos-10-cache-inventory-app --tags CLASS=ARCHIT --capabilities CAPABILITY_NAMED_IAM
```

##### 1: Test the faulty cache mechanism behaviour with the application.  What happens when you delete an Item and immediately refresh inventory?
- Get the url from the Stack
```
aws cloudformation --region $C9_REGION \
    describe-stacks \
    --stack-name archit-demos-10-cache-inventory-app \
    --query "Stacks[].Outputs[?OutputKey=='AppInstancePublicDNS'].[OutputValue]" \
    --output text
```
- Test inventory update & delete scenarios and look at _./artifacts/inventory-app-memcache/show-data.php_ to see the logic flaw in this read ahead cache scenario

---------------------------------------------------------------
---------------------------------------------------------------
#### CLEANUP
Only run these scripts if you are done cleaning &/or running all dependent demos.
```
aws cloudformation --region $C9_REGION delete-stack --stack-name archit-demos-10-cache-inventory-app
```
