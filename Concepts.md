## **EC2**
Elastic Computer Cloud - This is the machine that will run the OS for the application.

### Classes
- **T** - Use **CPU Credits**
- **P** - Used for graphic accelereted tasks.
- **M** - Can be more expensive than other, not use CPU credits.
- **R** - Memory optimized.

### Actions
- **Stopping instance** removes the charge of the service, but keeps the charge of space on disk (which is minimal).
- **Restarting instance** shut down the service and starts it on another machine. Can resolve problems with the virtual machine.
- Actions->Image and Templates->**Create Image** creates an image of the disk. **Note** selecting no reboot can make a problem with the image because of the file system that can be in use. On the other hand the reboot will cause a downtime.

## **EBS**
Elastic Block Store 
- It is a hard drive that is connected directly to your application.
- It can't be shared between applications.
- It can be resized according to the needs. Created additionally and mounted. (Like second hdd on pc)
- The images of ESB located under ELASTICK BLOCK STORE.
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
- Public urls to files, independant to the server.
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
    - add server acess logging
    - add encryption 
    - more...
- Permissions tab to edit access to the bucket globally. This is super sensitive!

## **S3 GLACIER**
We can move unused data(old data that we dont want to delete) automatically(lifecycle) to this storage type.
- It is cheaper than S3 (fraction of the mounthly fee of S3)
- Can take few hours before the data is accessible when working with the glacier! (like freezer)
- Lifecycle can be made via Managment tab in the S3 server.

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
    Allows any ip from outside to connect to port 22. No 80 port are enabled, hence we cant see the webpage, althougt we installed the apache2.

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
- **NAT Gateway** - servers can talk to outside world and get responses the same way "throught a door opened by the server". But this door cannot be opened from the outside world.
    - Nat gateway gives us elastic IP, like the **ISP** ip for our home internet.
- **Internet Gateway**, is like port forwarding on the router.
- **Elastic IP** - When shutting down EC2 and putting it up again the ip will change. 
    - We can request a pool of elastic IPs that will be reassigned to the instance after a restart.
    - It will cost monthly to keey this pool (because there are less and less IPv4 out there...)
- **AWS CLIENT VPN** - Allows us to connect to VPC to manage our DB for example.
- **AWS Site-to-Site VPN** - bridge my local network with a VPC.

## **ELB**
Elastic Load Balancer
- NLB - Network Load Balancer (good for streaming)
    - Faster (not looking closely at the trafic that comming in)
    - Less features
    - OSI Layer 4 routing
- ALB - Application Load Balancer
    - HTTP Routing rules (will look at the incoming traffic and rout based on a set of rules)
    - Most common elb
    - OSI Layer 7 routing

## **ROUTE 53 DNS** 
- Easy to remember - all dns servers are working on port 53.
- Not simple as regular dns (name - ip).
- Has additional features and routing. (Scailability and Up time of our app).
- We can set the DNS to point to the load balancer. When there are two settings - name.com and http.name.com, they can point to each other to reuse settings.
- DNS refresh in 15 min to 3 hours.

## **DBaaS**
Database as a Service
- The cloud provider manages the database servers and backups. You read and write your data to the manager service.
- If we join AWS with a working DB we can look at **Database Migration Service** for help.

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