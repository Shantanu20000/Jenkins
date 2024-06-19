# SonarQube server will require 3GB+ RAM to work effeciently 

# Install Database 
```
rpm -ivh http://repo.mysql.com/mysql57-community-release-el7.rpm
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
yum install mysql-server -y
systemctl start mysqld
systemctl enable mysqld
grep 'temporary password' /var/log/mysqld.log
mysql_secure_installation 
```
# Install Java 
```
yum install wget epel-release -y
yum install java -y
wget https://download.bell-sw.com/java/11.0.4/bellsoft-jdk11.0.4-linux-amd64.rpm
rpm -ivh bellsoft-jdk11.0.4-linux-amd64.rpm 
```
# To change java Version (java-11 required for student-application)
```
alternatives --config java 
```

# Configure Linux System for Sonarqube
```
echo 'vm.max_map_count=262144' >/etc/sysctl.conf
sysctl -p
echo '* - nofile 80000' >>/etc/security/limits.conf
sed -i -e '/query_cache_size/ d' -e '$ a query_cache_size = 15M' /etc/my.cnf
systemctl restart mysqld 
```
# Configure Database for Sonarqube 
```
mysql -p -u root -pRedhat@123
create database sonarqube;
create user 'sonarqube'@'localhost' identified by 'Redhat@123';
CREATE USER IF NOT EXISTS 'sonarqube'@'localhost' IDENTIFIED BY 'Redhat@123';
flush privileges;
```
# Install Sonarqube 
```
yum install unzip -y
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.1.zip
unzip sonarqube-7.9.1.zip
mv sonarqube-7.9.1 /opt/sonar
cd /opt 
```

# Configure Sonarqube 
```
sed -i -e '/^sonar.jdbc.username/ d' -e '/^sonar.jdbc.password/ d' -e '/^sonar.jdbc.url/ d' -e '/^sonar.web.host/ d' -e '/^sonar.web.port/ d' /opt/sonar/conf/sonar.properties
sed -i -e '/#sonar.jdbc.username/ a sonar.jdbc.username=sonarqube' -e '/#sonar.jdbc.password/ a sonar.jdbc.password=Redhat@123' -e '/InnoDB/ a sonar.jdbc.url=jdbc.mysql://localhost:3306/sonarqube?useUnicode=true&characterEncoding=utf&rewriteBatchedStatements=true&useConfigs=maxPerformance' -e '/#sonar.web.host/ a sonar.web.host=0.0.0.0' /opt/sonar/conf/sonar.properties
useradd sonar
chown sonar:sonar /opt/sonar/ -R
sed -i -e '/^#RUN_AS_USER/ c RUN_AS_USER=sonar' /opt/sonar/bin/linux-x86-64/sonar.sh 
```
# Start Sonarqube 
```
/opt/sonar/bin/linux*/sonar.sh start
/opt/sonar/bin/linux*/sonar.sh status
```
# Copy Public IP of Instance
```
<public_ip>:9000
```
# When we login first time sonarqube 

username = admin
password = admin

# Create Project
1. Fill required field.
2. Bellow output you will get 
mvn sonar:sonar \
  -Dsonar.projectKey=admin \
  -Dsonar.host.url=http://13.134.132.142:9000 \
  -Dsonar.login=5531f58cbf04a88a029eb50cfbf2728f78fsa3f4af 'Token'
# This is not best practice for Show your Credention on Pipeline file that why we take following step.
1. Copy our Token and go to Jenkins Server and Create global_Key
2. Open Jenkins Manage > Click on Credentials > Sellect Text File > Paste our Token in feild of Secret > Give ID name > Save and Create
3. Go to Job and Configured it Syantax Creator
4. Sellect WithSonarQube option and fill required fields
5. Generate syntax
   ```
   withSonarQubeEnv(installationName: 'sonar', credentialsId: 'sonar') {
                    sh '/opt/maven/bin/mvn sonar:sonar'
   }
   ```
7. Which we mention in Pipeline.
```
pipeline {
    agent any
    stages {
        stage('Pull') {
            steps {
                git 'https://github.com/Shantanu20000/studentapp-ui-jenkin-mvn.git'
                echo 'Pull Successfully'
            }
        }
        stage('Build') {
            steps {
                sh '/opt/maven/bin/mvn clean package'
                echo 'Build Successfully'
            }
        }
        stage('Test') {
            steps {
                withSonarQubeEnv(installationName: 'sonar', credentialsId: 'sonar') {
                    sh '/opt/maven/bin/mvn sonar:sonar'
                }
                echo 'Test Successfully'
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                echo 'Quality Gate Successfully'
            }
        }
        stage('Deploy') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'linux', path: '', url: 'http://172.31.36.228:8080')], contextPath: '/', war: '**/*.war'
                echo 'Deploy Successfully'
            }
        }
    }
}
```

   
