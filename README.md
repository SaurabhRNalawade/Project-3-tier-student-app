# ${\color{blue}\textbf{Project-3-tier-student-app}$

### Prerequisite:
- VPC
- Subnets
- Route Table
- Nat Gateway
- Internet Gateway
- RDS
### Create VPC 
- Name: VPC-3-tier
- CIDR: 192.168.0.0/16
  
### Create Subnets
1.Subnet-1
- Name: Public-Subnet-Nginx
- CIDR: 192.168.1.0/24
  
2.Subnet-2
- Name: Private-Subnet-Tomcat
- CIDR: 192.168.2.0/24
  
3.Subnet-3
- Name:Private-Subnet-Database
- CIDR: 192.168.3.0/24

### Create Internet Gateway 
- Name: IGW-3-tier
- attach igw to vpc
  
### Create Nat Gateway 
- Name: NAT-3-tier
- create in public subnet
  
### Create Route Table 
#### 1.RT-Public-Subnet
- add public subnet
- add igw
#### 2.RT-Private-Subnet
- add private subnets
- add nat
#### 3.RT-Private-Subnet
- add private subnets
- add nat

![image](https://github.com/user-attachments/assets/2d1b6f40-5582-4938-a06c-aa039d740a76)

### Create EC2 Instances 
1.Nginx-Server-Public ->create in public subnet ->allow port = 80,22

2.Tomcat-Server-Private ->create in private subnet ->allow port = 8080,22

3.Database-Server-Private ->create in private subnet ->allow port = 3306,22

![image](https://github.com/user-attachments/assets/387a3ea5-a26f-4de2-a1c7-cb31a6b06e02)

### Create Database In RDS 
- Go To RDS
- Created Database
- Standard create
- Free tier
- DB name – database-1
- Username – admin
- Password – Passwd123$
- VPC – VPC-3-tier
- Connect to Instance -> choose database instance
- Public access – no
- A.Z. – no preference
- Create database
- Edit security group -> Add 3306 port

![image](https://github.com/user-attachments/assets/342bae9f-595f-42d5-9f4d-11560244a041)

### Connect To Nginx-Server-Public 

![image](https://github.com/user-attachments/assets/e0853bf1-a8ad-4486-bfc0-7af7375af3a6)

- connect to instance
- change hostname
````
sudo -i
````
````
yum install nginx -y
````
- create file with name 3-tier-key.pem

````
vim 3-tier-key.pem
````
- copy private key and paste it here
  
### Now SSH into Database Server 

![image](https://github.com/user-attachments/assets/de5582a8-ffa7-4264-bd8d-c863baf33d50)
````
sudo -i
````
````
yum install mariadb105-server -y
````
````
systemctl start mariadb
````
````
systemctl enable mariadb
````
### Log in into database

![image](https://github.com/user-attachments/assets/4d359e49-f256-4b0f-8007-941099ba3122)
````
mysql -h rds-endpoint   -u admin -pPasswd123$
````
- Note: replace rds-endpoint with actual endpoint value
````
show databases;
````
````
create database  studentapp;
````
````
use studentapp;
````
### Run this query to create table:
````
 CREATE TABLE if not exists students(student_id INT NOT NULL AUTO_INCREMENT,  
	student_name VARCHAR(100) NOT NULL,  
	student_addr VARCHAR(100) NOT NULL,   
	student_age VARCHAR(3) NOT NULL,      
	student_qual VARCHAR(20) NOT NULL,     
	student_percent VARCHAR(10) NOT NULL,   
	student_year_passed VARCHAR(10) NOT NULL,  
	PRIMARY KEY (student_id)  
);
````
````
show tables;
````
- Logout from database:
````
exit
````
- back to nginx-server-public
### Now SSH into Tomcat Server 

![image](https://github.com/user-attachments/assets/572a481c-9182-4eb2-b821-0ef26d8e94f6)

- ssh -i 3-tier-key.pem ec2-user@ip-of-tomcat-vm
````
sudo -i
````
````
yum install java -y
````
````
mkdir /opt/tomcat
````
````
curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.90/bin/apache-tomcat-9.0.90.tar.gz
````
````
tar -xzvf apache-tomcat-9.0.90.tar.gz -C /opt/tomcat
````
````
cd /opt/tomcat/apache-tomcat-9.0.90.tar.gz/webapps
````
````
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/student.war
````
````
cd ../lib
````
````
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
````
 ### MODIFY context.xml:
 ````
cd apache-tomcat-9.0.90.tar.gz/conf
````
````
vim context.xml
````
- add below line [connection string] at line 21
````
 <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="USERNAME" password="PASSWORD" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://DB-ENDPOINT:3306/DATABASE-NAME"/>
````
````
cd ../bin
````
````
chmod +x catalina.sh
````
````
./catalina.sh start
````
````
yum install elinks -y
````
````
elinks
````
- using elinks package you can see output in cli use #elinks -> paste tomcat-ip:8080/student
````
exit
````
- back to nginx-server
````
vim /etc/nginx/nginx.conf
````
- set nu (enter below data in line 47 in between error and location)
````
location / {
proxy_pass http://private-IP-tomcat:8080/student/;
}
````
- wq ->save file
````
systemctl restart nginx
````
### Go To Browser Hit Public-IP Nginx

![image](https://github.com/user-attachments/assets/68631aed-182c-4ca7-b179-f23723dcea5c)

![image](https://github.com/user-attachments/assets/bcd68295-7c46-4332-85c6-f19270f0ee95)


