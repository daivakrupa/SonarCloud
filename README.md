# SonarCloud

SonarQube Installation and Integration with Jenkins

SonarQube is an open-source platform used for continuous analysis of source code quality by performing analysis of code to detect duplications, bugs, security vulnerabilities and code smells.

Step1: Install Java

sudo yum install java-1.8*

Java -version


Step2: Set Java path

Find where the java is installed

find /usr/lib/jvm/java-1.8* | head -n 3

Create a file vi /etc/profile.d/jdk.sh and paste the below lines
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/
PATH=$JAVA_HOME/bin:$PATH

save and exit
Reload profile.d/jdk.sh
source /etc/profile.d/jdk.sh
check the java Path
echo $JAVA_HOME


Step3: Install and configure mysql

wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm

sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm

sudo yum install mysql-server

Once installed start check the status of the mysql service
sudo systemctl start mysqld
sudo systemctl status mysqld

Configure mysql db by running mysql_secure_installation
mysql_secure_installation


Step 4: Create db and user for SonarQube

Login into mysql server
mysql -u root -p

Create db with below script
CREATE DATABASE sonarqube_db;
CREATE USER 'sonarqube_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON sonarqube_db.* TO 'sonarqube_user'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
Exit

Step 5: Install and configure SonarQube

Create a new user and set password for SonarQube

sudo useradd sonarqube
sudo passwd sonarqube

Download the latest version of SonarQube from the URL
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-6.7.7.zip

Unzip and rename it
 sudo unzip sonarqube-6.7.7.zip
 sudo mv sonarqube-6.7.7 sonarqube

Change the owner of the sonarqube directory
sudo chown -R sonarqube:sonarqube sonarqube

Open the sonarqube configuration file for changes
sudo vi /u01/sonarqube/conf/sonar.properties
Enter the database details below
sonar.jdbc.username=sonarqube_user
sonar.jdbc.password=password

sonar.jdbc.url=jdbc:mysql://localhost:3306/sonarqube_db?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance

add below entry in sonar.properties file
RUN_AS_USER=sonarqube
Save and Exit
Now configure SonarQube as a systemd service
Create a sonar.service file in system
sudo vi /etc/systemd/system/sonar.service

Add the below script
[Unit]
Description=SonarQube service
After=syslog.target network.target
[Service]
Type=forking
ExecStart=/u01/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/u01/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
[Install]
WantedBy=multi-user.target

Now start and check the status of the sonar service
sudo systemctl start sonar
sudo systemctl status sonar
sudo systemctl enable sonar

check the sonar using URL http://ip_address:9000
The default username and password of SonarQube is admin and admin.

Step 6: Integration with Jenkins Server
Install Git in your system

sudo yum install git -y

Install SonarQube scanner

sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.0.0.1744-linux.zip

Unzip the file and rename it
sudo uzip sonar-scanner-cli-4.0.0.1744-linux.zip

sudo mv sonar-scanner-cli-4.0.0.1744-linux.zip sonar-scanner

Set SonarQube sever details in sonar scan property file
sudo vi /u01/sonar-scanner/conf/sonar-scanner.properties

uncomment the sonar.host.url and replace localhost with sonarqube server ip
save and exit
Now login to your Jenkins server GUI and install SonarQube scanner plugin
Navigate to Manage Jenkins > Manage Plugins > Available > SonarQube scanner
Check the SonarQube Scanner and Install without Restart

Configure SonarQube server name and authentication token

Navigate to Manage Jenkins > Configure Systems > SonarQube Servers
Add Name =sonarserver
Server URL = http://ip_of sonarserver:9000
Login to SonarQube server as an admin My Account > Security > Generate Token
write token name and click Generate

Goto Global tool configuration > sonarqube scanners > save it.
Create a pipeline then goto   Build > add few lines 

        Sonar.ProjectKey="anyname"
        Sonar.ProjectName="anyname"
        Sonar.ProjectVersion=1.0
        Sonar.Sources=/var/lib/jenkins/workspace/$JOB_NAME/src

Then build it.

