# AWS ECS Deep Dive Section 3

# For AWS Linux
1. Install docker and give the proper access
2. Install git
```bash
$ sudo yum update -y
$ sudo amazon-linux-extras install docker -y
$ sudo service docker start
$ sudo usermod -a -G docker ec2-user
$ sudo yum install git -y
```
3. Setup CodeCommit with aws configure and git
```bash
$ aws configure
AWS Access Key ID [None]: Type your target AWS access key ID here, and then press Enter
AWS Secret Access Key [None]: Type your target AWS secret access key here, and then press Enter
Default region name [None]: Type a supported region for CodeCommit here, and then press Enter
Default output format [None]: Type json here, and then press Enter
$ git config --global credential.helper '!aws codecommit credential-helper $@'
$ git config --global credential.UseHttpPath true
$ git clone https://git-codecommit.us-east-2.amazonaws.com/v1/repos/MyDemoRepo my-demo-repo
```

#### Host 2 tier application using docker

* Pull the mysql image
```bash
$ docker pull mysql
```

# How to store data
* [The url with the official docs](https://hub.docker.com/_/mysql)
* Best practice to specify the version

1. Create a folder(in my case is ecsDocker)
2. Mount the folder and run docker
```bash
$ docker run --name some-mysql -v /Users/lbelonias/Desktop/untitled_folder/ecsDocker:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql
```
3. SSH to the container to check if everything is fine
```bash
$ docker exec -it some-mysql bash
```
4. Inside the container
```bash
$ whoami
$ mysql -u root -p
$ Enter password: my-secret-pw
```
5. Inside the MYSQL SHELL
```bash
$ SHOW DATABASES;
$ CREATE DATABASE createdByIakovosDB;
$ USE createdByIakovosDB
$ CREATE TABLE IF NOT EXISTS tasks (
    task_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    start_date DATE,
    due_date DATE);
$ DESCRIBE tasks;
$ SHOW TABLES;
$ INSERT INTO tasks (title, start_date, due_date) VALUES ('DockerAndKubernetes','2019-09-01','2019-10-1');
$ INSERT INTO tasks (title, start_date, due_date) VALUES ('AWSANDECS','2019-11-01','2019-12-1');
$ INSERT INTO tasks (title, start_date, due_date) VALUES ('EKSANDECR','2020-01-01','2020-02-1');
$ SELECT * FROM tasks;
$ CREATE DATABASE awsecs;
$ USE awsecs;
$ CREATE TABLE Employee (emp_id integer, first_name varchar(40), last_name varchar(40), primary_skills varchar(20), location char(10));
$ SHOW TABLES;
```
6. Stop the container and run it again to check if everything is ok
```bash
$ docker inspect CONTAINER_ID
$ docker stop CONTAINER_ID
$ docker run --name some-mysql -v /Users/lbelonias/Desktop/untitled_folder/ecsDocker:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql
$ $ mysql -u root -p
$ Enter password: my-secret-pw
$ USE createdByIakovosDB
$ SELECT * FROM tasks;
```
7. In case we need to create or restore sql db dumbs
```bash
$ docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /Users/lbelonias/Desktop/untitled_folder/ecsDocker/all-databases.sql

$ docker exec -i some-mysql sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < /Users/lbelonias/Desktop/untitled_folder/ecsDocker/all-databases.sql
```

# Flask preparation
1. [Get your project setup(tempates folders etc.)](https://github.com/Belonias/flaskToys)
2. Build each app image
```bash
$ docker image build -t addemp ./
$ docker image build -t getemp ./
```

3. Check the MySQL container IP
```bash
$ docker inspect some-mysql
```
4. In my case it was "IPAddress": "172.17.0.2", or use the name of your db container setup
5. Then run the flask images with proper dbhost
```bash
$ docker run -d -e DBHOST="some-mysql" -e DBPORT="3306" -e DBUSER="root" -e DBPWD="my-secret-pw" -e DATABASE="createdByIakovosDB" --name addem-v14 -p 8080:8080 addemp

$ docker run -d -e DBHOST="172.17.0.2" -e DBPORT="3306" -e DBUSER="root" -e DBPWD="my-secret-pw" -e DATABASE="createdByIakovosDB" --name addem-v17 -p 3000:80 addemp

$ docker run -d -e DBHOST="some-mysql" -e DBPORT="3306" -e DBUSER="root" -e DBPWD="my-secret-pw" -e DATABASE="createdByIakovosDB" --name getemp-v14 -p 80:80 getemp

$ docker run -d -e DBHOST="172.17.0.2" -e DBPORT="3306" -e DBUSER="root" -e DBPWD="my-secret-pw" -e DATABASE="createdByIakovosDB" --name getemp-v16 -p 80:8080 getemp
```

