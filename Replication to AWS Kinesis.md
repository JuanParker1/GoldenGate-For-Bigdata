# Lab 2 -  Oracle to AWS Kinesis Stream

## Before You Begin
-Create a Kinesis data stream(not included under Free-tier)on your AWS Instance, Follow the link for reference-
https://docs.aws.amazon.com/streams/latest/dev/learning-kinesis-module-one-create-stream.html

-It is strongly recommended that you do not use the AWS account root user or ec2-user for your everyday tasks, even the administrative ones. You need to create a new user with access key & secret_key for AWS, use the following link as reference to do the same :
           https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html

-Attach the following policies to the newly created user to allow access and GET/Put Operations on Kinesis data stream:
AWSLambdaKinesisExecutionRole-Predefined Policy in AWS.

-You need to attach the following inline policy as json:
```
"Version": "2012-10-17",
 "Statement": [
   {
   "Effect": "Allow",
     "Action": "kinesis:*",
     "Resource": [
       "arn:aws:kinesis:<your-aws-region>:<aws-account-id>:stream/<kinesis-stream-name>"
     ]
   }
```-


### Introduction
 This lab will guide through the steps required to start a replication of data between an Oracle database and AWS Kinesis Stream.

### Objectives
- Extract from Oracle to generate the Trail Files on Source
- Dump the trail files from Source to target machine
- Replicate from trail files on the target machine to Kinesis Stream.

### Time to Complete
Approximately 30 minutes


### What Do You Need?
You will need:
- Putty if you are using windows machine

In the last Lab, we learnt how to extract data from Oracle Database and write it Trail Files using GoldenGate. We will be using the same trail files to replicate the same data to AWS Kinesis Stream as well.

Now we will be working on the Target machine.We have a trail file created in the GGBD home in the /dirdat/oraTrails directory with the name tr. We will be using this trail file to send to a Kinesis Stream.

1. Copy the kinesis.props & kinesis.prm file to dirprm folder in your Golden Gate installation folder in your target machine from ./dirprm Directory.
![](/images/kineisis_1.PNG)
```
[oracle@gg4bd-target01 ggbd_home1]$ cd dirprm
[oracle@gg4bd-target01 dirprm]$ vi kinesis.props
```
Copy paste the below text in kinesis.props.


## KINESIS properties for Kafka Topic apply

```
gg.handlerlist=kinesis
gg.handler.kinesis.type=kinesis_streams
gg.handler.kinesis.mode=op
gg.handler.kinesis.format=json
gg.handler.kinesis.region=<your-aws-region>
#The following resolves the Kinesis stream name as the short table name
gg.handler.kinesis.streamMappingTemplate=<Kinesis-stream-name>
#The following resolves the Kinesis partition key as the concatenated primary keys
gg.handler.kinesis.partitionMappingTemplate=QASOURCE
#QASOURCE is the schema name used in the sample trail file
gg.handler.kinesis.deferFlushAtTxCommit=true
gg.handler.kinesis.deferFlushOpCount=1000
gg.handler.kinesis.formatPerOp=true
#gg.handler.kinesis.proxyServer=www-proxy-hqdc.us.oracle.com
#gg.handler.kinesis.proxyPort=80
goldengate.userexit.writers=javawriter
javawriter.stats.display=TRUE
javawriter.stats.full=TRUE
gg.log=log4j
gg.log.level=DEBUG
gg.report.time=30sec
gg.classpath=<path-to-your-aws-java-sdk>/aws-java-sdk-1.11.429/lib/*:<path-to-your-aws-java-sdk>/aws-java-sdk-1.11.429/third-party/lib/*e
##Configured with access id and secret key configured elsewhere
javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=ggjava/ggjava.jar
##Configured with access id and secret key configured here
javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=ggjava/ggjava.jar -Daws.accessKeyId=<access-key-of-new-created-user> -Daws.secretKey=<secret-ke-new-created-user>

```
Save the text using wq!

2. Edit the Replicat Parameter file to include the schema_name & table_name that needs to be replicated.
```
REPLICAT kinesis
-- Trail file for this example is located in "AdapterExamples/trail" directory
-- Command to add REPLICAT
-- add replicat kinesis, exttrail AdapterExamples/trail/tr
TARGETDB LIBFILE libggjava.so SET property=dirprm/kinesis.props
REPORTCOUNT EVERY 1 MINUTES, RATE
GROUPTRANSOPS 1
MAP pdb1.employees.economic_entity, TARGET oracle.*;
```
3. Add the replicat with the below command.
```
add replicat kinesis, exttrail ./dirdat/oraTrails/tr
```
4. Crosscheck for kinesis replicat’s status, RBA and stats.
Once you get the stats, you can view the kinesis.log from. /dirrpt directory which gives information about data sent to kinesis data stream and operations performed.
![](/images/kinesis_log[1].png)

