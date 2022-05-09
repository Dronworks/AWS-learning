## **EC2**
Elastic Computer Cloud - This is the machine that will run the OS for the application.

### Classes
- **T** - Use **CPU Credits**
- **P** - Used for graphic accelerated tasks.
- **M** - Can be more expensive than other, not use CPU credits.
- **R** - Memory optimized.

### Actions
- **Stopping instance** removes the charge of the service, but keeps the charge of space on disk (which is minimal).
- **Restarting instance** shut down the service and starts it on another machine. Can resolve problems with the virtual machine.
- Actions->Image and Templates->**Create Image** creates an image of the disk. **Note** selecting no reboot can make a problem with the image because of the file system that can be in use. On the other hand the reboot will cause a downtime.

### **EC2 Spot Instances**
Bidding on amazons unused capacity at a steep discount, but we will loose the spot to someone that will pay full price for the service. 

## **EBS**
Elastic Block Store 
- It is a hard drive that is connected directly to your application.
- It can't be shared between applications.
- It can be resized according to the needs. Created additionally and mounted. (Like second hdd on pc)
- The images of ESB located under ELASTIC BLOCK STORE.
- AMI - is the complete image of the app (good to snapshot frequently). When EBS only of the hdd.

## **EFS**
Elastic File System
- Mounted shared drive with several EC2 instances.
- It is slower than EBS. 
- It suppose to hold data, that need to be shared, i.e uploaded data, but for application we need faster storage like EBS.
- Installing EFS on EC2: 
    - Connect to the machine (sometime is will ask to use user ubuntu, can change it before @)
        ```
        ssh -i "awsdemo.pem" root@ec2-54-162-176-206.compute-1.amazonaws.com
        ```
    - Run **ADD** nfs support command:
        ```
        sudo apt-get install nfs-common
        ``` 
    - Create some folder for the share i.e on root
        ```
        cd /
        sudo mkdir efs
        ```
    - Mount efs: go to the efs and copy the dns - **fs-069fba947af37ac41.efs.us-east-1.amazonaws.com** and for the last part the name of the folder we created **efs**:  
        ```
        sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-069fba947af37ac41.efs.us-east-1.amazonaws.com:/ efs  
        ```
        **NOTE** the command will hang. This is because of the security group. We can add the *default* security group - that says that all ports and all protocols accepted for anybody using this security group id. **Instead of using ip range we can let access any aws resource who use this security group id**.
    - Now **do the same** on another instance and we would be able to access **/efs** from both instances and **share data**.

## **S3**
Simple Storage Service - Storage service (like dropbox)
- It is slower than both EBS and EFS.
- NO NEED for A SERVER to access files.
- Public urls to files, independent to the server.
- Names of the bucket should be unique, hence add some random numbers at the end of your name on creation.
- Default creation is like dropbox without internet access **but** with access from your app.
- BEST PRACTICE - use different buckets for public access and for sensitive data.
- Cli
    - **s3** commands - are more extensive commands directly with s3.
    - **s3api** commands - are api commands same as we use from code.
    - Upload file: **aws s3 cp ~/{file location} s3://{bucket name from ls command}/{bucket folder or just root}** i.e. aws s3 cp "~/Downloads/New Document(41).pdf" s3://dron123454/
    - aws also has sync commands to sync machine with a bucket.
- There is also different saving options for the files. Glacier or S3 single zone(not replicating data) or more. They are charged differently and have different time of access and different availability.
- In properties tab we can:
    - add versioning
    - add server access logging
    - add encryption 
    - more...
- Permissions tab to edit access to the bucket globally. This is super sensitive!

## **S3 GLACIER**
We can move unused data(old data that we don't want to delete) automatically(lifecycle) to this storage type.
- It is cheaper than S3 (fraction of the monthly fee of S3)
- Can take few hours before the data is accessible when working with the glacier! (like freezer)
- Lifecycle can be made via Management tab in the S3 server.

## **Cloud Front**
Replicating S3 data to servers, that are close to the customers (i.e Tokio)
- Great for static content. For example we use react on our web, this will let the users to see the web page faster, and less spinning wheel.

## **Security Group**
- Lets us access the application from outside (only the parts which are suppose to be accessed).
- **launch-wizard-1** - Default security group that was created on EC2 creation because of skipping steps.

    Example: **Inbound** RULE
    ```
    sgr-0b64e9158a2304d03	| 22	| TCP	| 0.0.0.0/0	| launch-wizard-1 
    ```
    Allows any ip from outside to connect to port 22. No 80 port are enabled, hence we cant see the webpage, although we installed the apache2.

    Example: **Outbound** RULE
    ```
    sgr-09725dbde7eecadf6	| All	| All	| 0.0.0.0/0	| launch-wizard-1
    ```
    Allows our server to reach any ip outside on any port.
    
- In Security group setting we can change the security types. For example change the access to port 22 only to my IP by selecting it from a drop down. And then something like this will appear: 192.114.151.1/32 (where 32 range = single ip)

## **VPC**
Virtual Private Cloud
- Servers can "talk" to each other without accessing the internet. 
- Example: two subnets, one with access to the internet and one with the DBases without access. Both are in the same VPC.
- **NAT Gateway** - servers can talk to outside world and get responses the same way "through a door opened by the server". But this door cannot be opened from the outside world.
    - Nat gateway gives us elastic IP, like the **ISP** ip for our home internet.
- **Internet Gateway**, is like port forwarding on the router.
- **Elastic IP** - When shutting down EC2 and putting it up again the ip will change. 
    - We can request a pool of elastic IPs that will be reassigned to the instance after a restart.
    - It will cost monthly to keep this pool (because there are less and less IPv4 out there...)
- **AWS CLIENT VPN** - Allows us to connect to VPC to manage our DB for example.
- **AWS Site-to-Site VPN** - bridge my local network with a VPC.

## **ELB**
Elastic Load Balancer
- NLB - Network Load Balancer (good for streaming)
    - Faster (not looking closely at the traffic that coming in)
    - Less features
    - OSI Layer 4 routing
- ALB - Application Load Balancer
    - HTTP Routing rules (will look at the incoming traffic and rout based on a set of rules)
    - Most common elb
    - OSI Layer 7 routing

## **ROUTE 53 DNS** 
- Easy to remember - all dns servers are working on port 53.
- Not simple as regular dns (name - ip).
- Has additional features and routing. (Scalability and Up time of our app).
- We can set the DNS to point to the load balancer. When there are two settings - name.com and http.name.com, they can point to each other to reuse settings.
- DNS refresh in 15 min to 3 hours.

## **DBaaS**
Database as a Service
- The cloud provider manages the database servers and backups. You read and write your data to the manager service.
- If we join AWS with a working DB we can look at **Database Migration Service** for help.
- If we already have a relational DB we can simply install it and use with EC2. But we will have to manage the Data and the backups etc...
- We can instead give AWS the DB and they will do all of this for us(**Aurora**). And it is called **RDS** (Relational Database Service).
    - Notice Best practice is to create small DB and scale up. You won't be able to scale down!!!
    - **Multy AZ** - multiple zones, and if you need to do changes one cluster will be available but with additional cost. In the future, it is possible to move to AZ, so start small if needed.
- **NO SQL DB** 
    - DynamoDB 
        - No management from user side
        - Easy global tables
        - Multi region load balancing
    - DocumentDB - is sort of MongoDB.
        - Although we can use mongo db as well.

## **PaaS** 
Platform as a Service - you just provide the sourcecode and the cloud manages the os and the server packages.
- Behind the management are still the networking components, the EC2 and all the other things.
- **Elastic Beanstack** - if you have a single code base, you give it to AWS and they build the ecosystem.
- **ECS** Elastic Container Service - control EC2 for you, no need to set up clusters and do server administration. We can focus on the containers and MS architecture.
    - Container can be WEB server and run on EC2
    - Container can be a simple job (like bash script), that closing after execution.

## **FaaS**
Function as a Service - each time some event occurs(like incoming request) a small chunk of code can run on the cloud.
- **Lambda** service running single execution of our code as many times as needed.
- **Serverless Architecture** - An application that responds to incoming event without the need for always-on servers that you manage. 
    - In the background it takes the source code, building a docker and running it on EC2.
- **AWS Batch** - for backend jobs.
- **Step Functions** - for multistep workflows. Steps through a series of tasks (like long bash code as Linux Cron Job).
- **SWF** Simple Workflow is more heavy than Step Function.

## **SaaS**
Software as a Service - software and app integrations that run in the cloud that you don't have to write or maintain (like Google Docs).
- **Cognito** - manages sign in with providers(your user can sign in with google or facebook). And do much more, such as calling lambdas when some one login.
- **API Gateway** - fully featured rest api - can send api once to your app and once to a lambda, etc...
- **AppSync** - for graphQL.
- **Amplify** - app framework for setting up components using cli commands.
- **SageMaker** AI - helps build and train machine learning models.
- **Comprehend** - give it a block of text and get back the mood of the writer.
- **Lex** - chat bot for the app.
- **Personalize** - promote specific products based on shopping habits.
- **Poly** - reads dynamic text with some pretty life-like voices.
- **Rekognition** -find faces in an image, or extract text from an image.
- **Textract** - processing a lot of documents.
- **Translate** - translating languages.
- **Transcribe** - like Alexa. Converts natural speech in to text.
- **MediaConvert** - take files from S3 and convert them into various media formats. Also optimize for devices.
- **AWS IoT Core** - support for smart devices.

## **DevOps** 
- **CI** Continuous Integration - Smaller changes to the code are automatically tested whenever a change is made by any developer.
- **CD** Continuous Deployment - Tested changes are automatically deployed to a staging environment and can then be automatically sent into production.
- **CodePipeline** - automatically pull the code and build your project and run some tests using **CodeBuild**. Also copdepipeline can use **CodeDeploy** for the CD part.
- **AWS OpsWorks** - work with Puppet, Chef, Ansible or AWS own infrastructure tool **CloudFormation**.
- **CloudWatch** - for metrics, memory usage, error messages in the application and more.
- **APM** app performance monitoring - checks if an app performing slow.

## **SECURITY**
- **WAF** Web Application Firewall - connect to the load balancer and can be used as other firewall products. Looks at a set of rules constantly published when new attacks discovered.
- **Shield** - helps for DDoS.
- **GuardDuty** - can find suspicious server connection that our server can be making. Looks if anything the server do is out of place.
- **Inspector** - like a full scan of your cloud infrastructure. Like full virus scan.
- **Macie** - looks through data and tells us if it looks like we sharing online sensitive data.
- **CloudTrail** - audit trail of changes made internally to our aws. Also can show us if someone uses cloud keys for incorrect usages.
- **SecurityHub** - puts all those products under one place.

## **ELASTIC CACHE**
Amazon management of cache services (you can install redis on your ec2 as well)
- Redis
- Memcached

## **DATA LAKE**
Storage of lots of data that is not sorted or organized. Gigabytes of data in an S3 bucket.

## **REDSHIFT**
Amazon management of big data
- Is a cluster of Postgres DB servers - If you have a lot of data that needed to be searched frequently...

## **EMR**
For apache spark or hadoop.

## **ATHENA** 
For bunch of text files in an S3 bucket (**which is a data lake**) and we need to do queries for this data.

## **QUEUES**
- **Kenesis** (like a nail gun) - Handling large stream of incoming data. Also connects other services to do real time reporting to the stream.
- **SQS (Simple Queue Service)** (like a hummer) - simple to use Queues but can get expensive it sending there lots and lots of events.
- **SNS (Simple Notification Service)** - push out a message (email/text message), for example after sqs finished generating a report sns can send an email that the report is ready to download.

## **AWS CLI**
Run commands of AWS on local machine.
- We can install it from **msi** for example.
- Check installation with aws --version
- To configure cli connection we need the key that we stored...
    ```
    aws configure
    <enter the id>
    <enter secret access key>
    <region of the (for example s3)>
    <default output format (for example json)>
    ```
- Testing configuration **aws s3api list-buckets** or **aws s3 ls** (same but not in default output format)

## **SDK**
We can do things on server via SDK (for example creating a demo.txt on S3). But to be able to access the server we need the access key.
- First example was to run the code with the keys in the script file. THIS IS BAD!
- We can use roles for avoid using keys!

## **ROLE**
Roles can be applied to an instance and then to access instance no key required.
- By default we can give a role to **ALL S3**s.
- Also we can give a role to a **specific S3**.
- When we are connected to the server the ROLE identifies our IAM and we can run scripts without the key.