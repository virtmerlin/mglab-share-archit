### 02-s3-multi-part-upload
##### GIVEN:
  - An AWS Cloud9 desktop from 01-setup-cloud9
  - A rather large media file that I want to upload to a website

##### WHEN:

  - I use the aws cli to 'multi-part-upload' it

##### THEN:
  - I will be able to upload it faster

##### SO THAT:
  - I can learn how to use s3 multipart uploads

##### NOTE: _This demo will provide CLI steps, but may have been demonstrated via the UI during a class delivery_

##### [Return to Main Readme](https://github.com/virtmerlin/mglab-share-archit#demos)

---------------------------------------------------------------
---------------------------------------------------------------
#### REQUIRES
- 01-setup-cloud9
- 02-s3-setup-simple-webapp

---------------------------------------------------------------
---------------------------------------------------------------
#### DEMO

##### 1: Upload large media file for the car website
[DOC-LINK:s3-multipart-upload-cli](https://aws.amazon.com/premiumsupport/knowledge-center/s3-multipart-upload-cli/)
- Reset your region & AWS account variables in case you launched a new terminal session
```
cd ~/environment/mglab-share-archit/demos/02-s3-multi-part-upload/
export C9_REGION=$(curl --silent http://169.254.169.254/latest/dynamic/instance-identity/document |  grep region | awk -F '"' '{print$4}')
echo $C9_REGION
export C9_AWS_ACCT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep accountId | awk -F '"' '{print$4}')
echo $C9_AWS_ACCT
```
- Copy a large video file down to Cloud9 so we can upload it to our bucket
```
mkdir mywebsite-v2 && cd mywebsite-v2 && wget -c https://mglab-aws-samples.s3.amazonaws.com/classes/archit/02/archit-s3-car-site-v2.tgz -O - | tar -xz && cd ..
ls -allh ./mywebsite-v2/videos/*
```
- Get the Bucket name created from demo 02-s3-setup-simple-webapp
```
BUCKET_NAME=$(aws resourcegroupstaggingapi get-resources --region $C9_REGION \
   --tag-filters 'Key=DEMO,Values=02-s3-multi-part-upload' \
   --query ResourceTagMappingList[].ResourceARN | grep "arn:aws:s3" | tr -d '"' | tr -d ' '| awk -F ':' '{print$6}')
echo $BUCKET_NAME
```
- Open a second terminal in the Cloud9 IDE and watch for multipart uploads
```
BUCKET_NAME=$(aws resourcegroupstaggingapi get-resources --region $C9_REGION \
   --tag-filters 'Key=DEMO,Values=02-s3-multi-part-upload' \
   --query ResourceTagMappingList[].ResourceARN | grep "arn:aws:s3" | tr -d '"' | tr -d ' '| awk -F ':' '{print$6}')
echo $BUCKET_NAME
watch aws s3api list-multipart-uploads --bucket $BUCKET_NAME
```
- Start the multipart upload in the original terminal and 'watch' the polling command from above ... you may need to run more than once to see it show up on the second terminal.
```
aws configure set default.s3.max_concurrent_requests 20
aws s3 cp ./mywebsite-v2/videos/classic.mp4 s3://$BUCKET_NAME/videos/
```

##### 2: Enable Bucket versioning so we can update the index.html and roll back if we need to
- Enable Bucket versioning
```
aws s3api put-bucket-versioning --bucket $BUCKET_NAME --versioning-configuration Status=Enabled
```
- Copy the new index file to the bucket so the website will play media now
```
aws s3 cp ./mywebsite-v2/index.html s3://$BUCKET_NAME
aws s3api list-object-versions --bucket $BUCKET_NAME --prefix index.html
```

##### 3: See if our video enabled website is up :)
- Mopar or No car !!!!
```
echo "http://$BUCKET_NAME.s3-website-$C9_REGION.amazonaws.com"
```

#### CLEANUP
- Uses CLEANUP from 02-s3-setup-simple-webapp
