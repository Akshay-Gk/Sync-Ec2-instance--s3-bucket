# Sync ec2 instance with S3 Bucket

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/1fe57e35-638a-4918-b356-ff6a5b715cf5)


# Description:

In this article, we learn how to synchronize an entire Amazon S3 bucket to a local directory location of an ec2 instance. 

Amazon S3 is a repository for internet data. Amazon S3 provides access to reliable, fast, and inexpensive data storage infrastructure. It is designed to make web-scale computing easier by enabling you to store and retrieve any amount of data, at any time, from within Amazon EC2 or anywhere on the web

# Steps to achieve this:

* _Step 1 : Create an Ec2 instance_
* _Step 2 : Install Apache (httpd)_
* _Step 3 : Setup a website_
* _Step 4 : Copy extracted files_
* _Step 5 : Create S3 Bucket_
* _Step 6 : Make bucket Public_
* _Step 7 : Create IAM user_
* _Step 8 : Create Access Key_
* _Step 9 : Configure IAM user on ec2_
* _Step 10 : Sync /var/www/html/images/ to S3 Bucket_
* _Step 11 : Add HTTPD Rewrite Engine_

# Step 1 : Create an Ec2 instance

* Open the Amazon EC2 console, Choose Launch Instance
* Enter name 

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/cefe5844-6f3a-4bfe-81d6-ae5473993b4d)

* Choose an Amazon Machine Image (AMI), find an Amazon Linux AMI.

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/79261b83-7c9a-4691-9ac5-f61f86136143)

* Create or add a Key pair

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/1555f8e5-20fb-4796-909c-e735915b277d)

* Create a new security group or Add existing one as per need

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/0e749606-52a2-4c65-b87e-c716c852b6a2)

* Click "Launch instance"
 
* Now we can see created instance
 
![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/1712d61c-bb68-45c1-a8bf-ca6be0e25bf7)


# Step 2 : Install Apache (httpd)

* Use below command to install apache

```
sudo yum install httpd -y
sudo systemctl restart httpd.service
sudo systemctl enable httpd.service
```
```
sudo chown -R apache:apache /var/www/html/
```

# Step 3 : Setup a website 


> `Note: Here I'm using a sample website template from too plate (www.tooplate.com) to demonstrate the process`

* Download template file to ec2

```
wget https://www.tooplate.com/zip-templates/2132_clean_work.zip
```

* unzip the downloaded file

```
unzip 2132_clean_work.zip
```

# Step 4 : Copy extracted files


* Copy Extracted files to /var/www/html
```
sudo cp -r 2132_clean_work/* /var/www/html/
```

* Call Public DNS name of instance

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/c4117155-a537-477d-a648-223e590101d7)


> `Note: Note that the site is still loading from ec2. We setup the *"images"* in the site to be loading from s3 bucket`


# Step 5 : Create S3 Bucket

*A bucket is a container for objects stored in Amazon S3*

* In the left navigation pane, choose Buckets 
* Choose Create bucket

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/80817cd3-0ace-4041-a8db-8dcaaf09ebb6)

* Enter bucket name
* Choose Region

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/98016bed-e0f1-4f9f-990c-feff52d335a7)


* Disable "Block all public access"
> Inorder to make bucket public we should disable "Block all public access"

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/b86400d1-6331-474c-b0ac-ca8e9a5a40a1)

* Rest keep default setting
* Click "Create" 

 ![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/e314d568-305e-46ff-af1b-77a038083147)

**_Now we can see created s3 bucket_**

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/6ef5bf13-296f-408d-bcac-537fc8af9d0c)


# Step 6 : Make bucket Public


* Go inside the created bucket
* Go to "Permission" tab

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/66b94cfb-fe6d-4e74-9f5e-bf2f9fab2650)

##  *Add Bucket Policy*
*The bucket policy, written in JSON, provides access to the objects stored in the bucket. Bucket policies don't apply to objects owned by other accounts*

* Under "Bucket policy" click edit
* Add Bucket policy for public access

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:Getobject*",
            "Resource": [
                "arn:aws:s3:::mysite-learndevops",
                "arn:aws:s3:::mysite-learndevops/*"
            ]
        }
    ]
}
```

> Give your bucket arn in bucket policy

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/c3eaca2d-67ea-49e2-960d-7f38eedbf3ea)


# Step 7 : Create IAM user

*An AWS Identity and Access Management (IAM) user is an entity that you create in AWS. The IAM user represents the human user or workload who uses the IAM user to interact with AWS*

**_Create an IAM user with s3 full access_**

* On the Console Home page, select the IAM service

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/a5650f1e-fe80-42ae-afbd-779ce3f29e67)

* In the navigation pane, select Users and then select Add users

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/12bcca9c-5c93-4407-939c-55ac01262b10)

* **Create an user with **_"programmatic access"_****

* Enter name
* Click *"Next"*

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/d688efde-b29f-43e4-931e-aaadf59177d5)


* Choose **"Attach policies directly"**
* Under **"Permissions policies"** search *s3fullaccess* and Choose **_"AmazonS3FullAccess"_**
* Choose "Next"

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/81505929-3d76-4b16-b408-2121dc51e3ad)

_Now we can see the created user_

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/68aa4501-ede2-48c5-9da8-d0b9e0a5e133)


# Step 8 : Create Access Key

_Create Access key for "akshay" user_
* Click "Create access key"

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/7a1dc23c-6902-4ecc-8d6d-ac7bf21fbd37)

* Under **"Access key best practices & alternatives"** select **"Other"**
* Hit **"Next"** then click **"Create access key"**

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/456faaa1-0b8d-44ca-8169-bda9f2c4b7e6)

* Save the Access key and Secret Access key 

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/4465ec35-05e9-4f23-8913-f26bdfd402a0)


# Step 9: Configure IAM user on ec2


* SSH to ec2 instance using Public DNS name

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/b868a915-6840-40d6-adae-94e91415b27e)

* Enter below command to configure user
```
 aws configure --profile=akshay
 ```
 _Enter below details_
 1. Access key
 2. Secret access key
 3. Region : ap-south-1
 4. Output format : json

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/d9759f0a-17fd-4207-a102-90bc164de999)

* When i do list i'm able to see the s3 bucket

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/2528844d-31a4-4cad-bfa3-2ef6bc655fa6)

# Step 10 : Sync /var/www/html/images/ to S3 Bucket


Use command:
```
aws s3 sync /var/www/html/images/ s3://"bucket name" --profile="user"
```

* **Now we can find the Images inside /var/www/html/images/ is copied to S3 bucket**

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/5338566b-f7c9-4d36-adf7-6735bf5e57ee)



# Step 11 : Add HTTPD Rewrite Engine

_A rewrite engine is a component of web server software that allows you to rewrite or redirect uniform resource locators (URLs)_ 


### **_We need to add HTTPD "Rewrite Engine" in HTTPD conf file_**

Use command:
```
sudo vim /etc/httpd/conf/httpd.conf
```

* Rewrite Engine:


use command:
```
RewriteEngine On
RewriteRule ^/images/(.*)$ https://s3.ap-south-1.amazonaws.com/mysite-learndevops/$1 [L]
```

* **_Now site loads from Ec2_**

![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/b3b50a40-0c42-44c2-8faf-005a8ba3bd9e)




* **_Images loads from s3 bucket_**


![image](https://github.com/Akshay-Gk/Sync-Ec2-instance-s3-bucket/assets/112197849/22dd612b-96d6-41b4-9190-b10b3ec74319)



# Description

Here we have created an ec2 instance and hosted a demo website. We sync the images of the website with the s3 bucket we created. This model helps to store the data on the s3 bucket while we run our website on ec2.



