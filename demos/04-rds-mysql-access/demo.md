### 04-rds-mysql-access
##### GIVEN:
  - An AWS Cloud9 desktop from 01-setup-cloud9

##### WHEN:

  - I create a multi-az RDS Mysql Instance
  - I access it via the mysql cli & manually fail it over

##### THEN:
  - I will be able to access it still

##### SO THAT:
  - I can learn how multi-az Mysql works

##### NOTE: _This demo will provide CLI steps, but may have been demonstrated via the UI during a class delivery_

##### [Return to Main Readme](https://github.com/virtmerlin/mglab-share-archit#demos)

---------------------------------------------------------------
---------------------------------------------------------------
#### REQUIRES
- 01-setup-cloud9

---------------------------------------------------------------
---------------------------------------------------------------
#### DEMO

##### 1: Create CloudFormation 'Stack' that will create req'd security groups & a multi-az Mysql RDS instance.
- Reset your region & aws account variables in case you launched a new terminal session
```
cd ~/environment/mglab-share-archit/demos/04-rds-mysql-access/
export C9_REGION=$(curl --silent http://169.254.169.254/latest/dynamic/instance-identity/document |  grep region | awk -F '"' '{print$4}')
echo $C9_REGION
export C9_AWS_ACCT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep accountId | awk -F '"' '{print$4}')
echo $C9_AWS_ACCT
```
- Create 'Stack'
```
aws cloudformation deploy --region $C9_REGION --template-file ./artifacts/04-rds-mysql-access-cfn.yaml \
    --stack-name archit-demos-04-rds-mysql-access --tags CLASS=ARCHIT --capabilities CAPABILITY_NAMED_IAM
```

##### 2: Test connectivity with mysql
- Get database URL & Port to connect to it via a mysql client
```
DBENDPOINT=$(aws cloudformation --region $C9_REGION \
    describe-stacks \
    --stack-name archit-demos-04-rds-mysql-access \
    --query "Stacks[].Outputs[?OutputKey=='DBENDPOINT'].[OutputValue]" \
    --output text)
echo $DBENDPOINT
DBPORT=$(aws cloudformation --region $C9_REGION \
    describe-stacks \
    --stack-name archit-demos-04-rds-mysql-access \
    --query "Stacks[].Outputs[?OutputKey=='DBPORT'].[OutputValue]" \
    --output text)
echo $DBPORT
```
- Connect with Mysql ... Use _Passw0rd_ when prompted for one.
```
mysql -u admin -p -h $DBENDPOINT -P $DBPORT
```
- At the mysql prompt enter
```
CREATE USER IF NOT EXISTS 'fred'@'mg.lab' IDENTIFIED BY 'new_password';
SHOW DATABASES;
exit
```

##### 3: Simulate console failure with ping in cloud9
- In the Cloud9 terminal prompt
```
watch nslookup $DBENDPOINT
```
- In the RDS console: Goto 'Actions' & reboot the RDS instance 'with failover'
  - [AWS RDS Console](https://console.aws.amazon.com/rds/home) ... (you may need to select your region)
  
- Observe the DNS hostname resolution for the DB 'failover' to the secondary in the Cloud9 terminal

---------------------------------------------------------------
---------------------------------------------------------------
#### CLEANUP
Only run these scripts if you are done cleaning &/or running all dependent demos.
```
aws cloudformation delete-stack --region $C9_REGION  --stack-name archit-demos-04-rds-mysql-access
```
