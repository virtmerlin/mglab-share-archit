### 02-s3glacier-vault
##### GIVEN:
  - An AWS Cloud9 desktop from 01-setup-cloud9
  - A zip file with a versioned copy of my static website

##### WHEN:

  - I use the AWS cli to upload it to an S3Glacier vault

##### THEN:
  - I will be able to request a restore

##### SO THAT:
  - I can learn how to use s3 Glacier for backup archives

##### NOTE: _This demo will provide CLI steps, but may have been demonstrated via the UI during a class delivery_
##### NOTE: _This demo may take 'hours' to complete due to glaciers default archive policies ... you will need to be patient if you choose to do this one_

##### [Return to Main Readme](https://github.com/virtmerlin/mglab-share-archit#demos)

---------------------------------------------------------------
---------------------------------------------------------------
#### REQUIRES
- 01-setup-cloud9

---------------------------------------------------------------
---------------------------------------------------------------
#### DEMO

##### 1: Create an S3 Glacier vault
- Reset your region & aws account variables in case you launched a new terminal session
```
cd ~/environment/mglab-share-archit/demos/02-s3glacier-vault/
export C9_REGION=$(curl --silent http://169.254.169.254/latest/dynamic/instance-identity/document |  grep region | awk -F '"' '{print$4}')
echo $C9_REGION
export C9_AWS_ACCT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep accountId | awk -F '"' '{print$4}')
echo $C9_AWS_ACCT
```
- Create the vault
```
aws glacier create-vault --vault-name mglab-demo-archit --region $C9_REGION --account-id -
aws glacier add-tags-to-vault --region $C9_REGION --account-id - --vault-name mglab-demo-archit --tags CLASS=ARCHIT,DEMO=02-s3glacier-vault
```

##### 2: Pull down a version of the website to archive
- Wget that Mopar website to backup !!!
```
wget -c https://mglab-aws-samples.s3.amazonaws.com/classes/archit/02/archit-s3-car-site-v1.tgz
```

##### 3: Create a Glacier Archive ... but before you do ... look in the UI and notice there aren't any there yet.  It will take many hours for the Console to 'update' its Vault inventory after you create archives.
  - [DOC-LINK: Glaicer CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-glacier.html#cli-services-glacier-prep)
  - [DOC-LINK: Glacier Vaults](https://aws.amazon.com/premiumsupport/knowledge-center/cli-glacier-vault/)
  - [DOC-LINK: Glacier Initiate Jobs](https://docs.aws.amazon.com/amazonglacier/latest/dev/api-initiate-job-post.html)
  - [DOC-LINK: Gacaier Archives 3rd party Howto](https://softwaredevelopmentstuff.com/2017/05/02/downloading-an-aws-glacier-archive-step-by-step/)

- Lets upload a small archive so we don't have to multipart it ...
```
sudo yum install jq -y
ARCHIVE=$(aws glacier upload-archive --account-id - --vault-name mglab-demo-archit --region $C9_REGION --archive-description "demo website v1" --body archit-s3-car-site-v1.tgz)
ARCHIVE_ID=$(echo $ARCHIVE | jq .archiveId | tr -d '"')
echo $ARCHIVE_ID
```

##### 4: Now that we have an archive in our vault ... Lets retrieve it
- Start Inventory Job to get a listing of all archives, otherwise will only happen 1/day, but be patient as this may take an hour or 2
```
aws glacier initiate-job --account-id - --vault-name mglab-demo-archit --region $C9_REGION --job-parameters '{"Type": "inventory-retrieval"}'
```
- Poll the Inventory job status until  you see it has completed before continuing.  Be patient.  Could we put a notification to SNS to email us when its done?
```
aws glacier list-jobs --account-id - --vault-name mglab-demo-archit --region $C9_REGION
```
- Initiate the Archive retrieval job, might be a good idea to add a SNS notification to this as well as retrievals could take hours.
```
sed  -i "s/<ARCHIVE_ID>/$ARCHIVE_ID/g" ./artifacts/job-retrieval.json
aws glacier initiate-job --account-id - --vault-name mglab-demo-archit --region $C9_REGION --job-parameters file://./artifacts/job-retrieval.json
```
- Show job Status ... you may need to wait hours for both the inv & archive retrieval to finish.   When they eventually succeed, your vault will show the new archive in the console and be ready for restore.
```
aws glacier list-jobs --account-id - --vault-name mglab-demo-archit --region $C9_REGION
```
- Fetch the completed retrieval job ID & archive ID ... we will need a bit of jq & bash magic to fetch the job_id from json
```
sudo yum install jq -y
CMD="aws glacier list-jobs --account-id - --vault-name mglab-demo-archit --region $C9_REGION | jq -r '.JobList[] | select(.ArchiveId == \"$ARCHIVE_ID\") | .JobId'"
ARCHIVE_JOBID=$(eval $CMD)
CMD2="aws glacier list-jobs --account-id - --vault-name mglab-demo-archit --region $C9_REGION | jq -r '.JobList[] | select(.ArchiveId == \"$ARCHIVE_ID\") | .RetrievalByteRange'"
ARCHIVE_BYTES=$(eval $CMD2)
echo $ARCHIVE_JOBID
echo $ARCHIVE_BYTES
```
- Lets actually get our Archive back from Glacier now ...
```
aws glacier get-job-output --account-id - --vault-name mglab-demo-archit --region $C9_REGION --job-id $ARCHIVE_JOBID --range "bytes=$ARCHIVE_BYTES" my-restored.tgz
```

---------------------------------------------------------------
---------------------------------------------------------------
#### CLEANUP
Only run these scripts if you are done cleaning &/or running all dependent demos.
- [DOC-LINK: Glacier Cleaning up Archives](https://docs.aws.amazon.com/amazonglacier/latest/dev/deleting-an-archive-using-cli.html)

The command below havent yet been sorted into an order to be 'repeatable' , but following them you should be able to clean up that Glacier Vault by first deleting all archives via the cli, then using the console to delete the vault.  be patient (hours).

        CMD3="aws glacier list-jobs --account-id - --vault-name mglab-demo-archit --region $C9_REGION | jq -r '.JobList[] | select(.Action == \"InventoryRetrieval\") | .JobId'"
        ARCHIVE_INV_JOBID=$(eval $CMD3)

        echo $ARCHIVE_INV_JOBID

        aws glacier get-job-output --account-id - --vault-name mglab-demo-archit --region $C9_REGION --job-id $ARCHIVE_INV_JOBID my-inv.json

        *** Delete each archive in my-inv.json with aws glacier delete-archive

        aws glacier delete-vault --account-id - --vault-name mglab-demo-archit --region $C9_REGION
