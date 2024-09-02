# Deploy-High-Available-Architecture-in-AWS

In this Project, We create webserver and database server in aws and make them high available using autoscaling.


Step 1:
Create Vitual Private Cloud 
Go to aws vpc -> click create vpc -> select vpc only-> 10.0.0.0/16 as ipv4->create vpc

Step 2:
Setting up subnets in vpc:
Go to aws vpc -> click subnets -> create subnet->select your vpc->create 2 subnetes-> publuc subnet 10.0.0.0/24 and private subnet 10.0.1.0/24

Step 3:
Configuring internet access

To give internet for public, We need to create internet gateway
Go to aws vpc ->click IG->Create internet gateway

To route the internet from IG to public subnet, we need public route table.
Go to aws vpc->click route tables->create public route table

To attach IG to Routetable
Go to public route table->click edit route->add IG as 0.0.0.0/0

To attach public subnet to Routetable
Go to public routetable->click subnet associations ->click edit and attach public subnet

To give internet for private subnet, we need NAT Gateway that is not now needed. 
Now we need to attach private subnet to private routetable. But bydefault private riutetable is created when you create vpc.
Go to private routetable->click subnet associations->click edit and attach private subnet.

Step 4: 
Adding Security

for that we need to create security group
Go to security groups ->click create -> Select inbount rules as http and ssh

Step 5:
Launch ec2 instance on your public subnet

Go to aws ec2->launch ec2 instance -> select ami as amazon linux2->t2.micro as instance type->create newkeypair->slect vpc->select punlic subnet->select security group you created->click launch

Step 6:
Installing wordpress application on ec2 instance

Go to aws ec2->click your ec2 instance->connect ec2 using ec2 connect method.

Now you can execute linux commands on ec2.
Sudo youm update-> to check everyting is updated or not.
sudo yum install httpd -> to install apache webserver so that you can install application on it.
sudo systemctl start httpd ->to start apache server
sudo systemctl status httpd -> to check status of server

copy your ec2 public ip and paste it in browser like-> http:<public ip>   you can view apache server is running

sudo amazon -linux -extras install -y lamb -mariadb10.2 -php 7.2 php 7.2  -> to install mysql
sudo yum install mysql-> to  install mysql
cd /var/www/html
sudo wget https://wordpress.org/latest.tar.gz  -> to download wordpress
ls -> latest.tar.gz(your wordpress is download in zip format)
sudo tar -xzvf latest.tar.gz -> to unzip above file
sudo cp wp-config-sample.php wp -config.php  -> to copy 1st file into 2nd file
sudo chown -R apache:apache /var/www/html/wordpress -> to set permission for apache user
sudo chmod -R 755 /var/www/html/wordpress  -> to set permission for file

Step 7:
Creating database for wordpress application server

To create database server, we need subnet group.
Go to aws rds -> click create subnet group->select your vpc->select private subnet and region(because for database, we need to host inside private subnet)

but it will raise error. because you need to host database in 2AZ's. so we need to create another private subnet(ap-south-1b 10.0.2.0/24). 
So create another private subnet similar to which we create in step3 and add this private subnet in private routetable subnet association.Now your subnet group is created in RDS.

go to aws rds->click create database->select mysql->select template as freetier -> select vpc and 2 both private subnets->create new security group(type:mysql/aurora, protocol type:tcp,port:3306,source:webserver security group)-> create database

Step 8:
configure datatbase on ec2 instance

Go to aws ec2->connect ec2 instance by ec2 connect method
execute below commands
export MYSQL_HOST = <Database endpoint>   ( you can get Database endpoint  from datbase which is created in aws rds)
mysql --user=<username> --password=<password> wordpress         (you can get username,password while creating database in aws rds)
exit (because you are connected to mysql db. so exit fron that)
cd /var/www/html/wordpress
ll -> to see congig files in linux

Now we need to configure mysql db on wp config file.
sudo vi wp -config.php
modify thease details: dbname,username,password,endpoint
save the file
less wp -config.php(used to list the content of file one screen at a time)
copy the content inside the link and paste it on wp -config.php file. -->https://api.wordpress.org/secret-key/1.1/salt/
save the file

to allow w3tc plugin
sudo vi wp -config.php
add this line at the end of file -> define('W3TC_CONFIG_DATABASE',true);
save the file

to deploy wordpress
sudo yum install php -xml
cd..
pwd -> html
sudo cp -r wordpress/* /var/www/html
sudo chown -R apache:apache /var/www/html

to start hosting apache webserver
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
sudo systemctl restart php -fpm


Step 9:
Access our application server in webbrowser

Go to browser and type -> http://<public ip of ec2 instance>/wp-admin

Step 10:
Access appication in custom domain name

If you want access your application in custom domain name, you can register domain in route53.

