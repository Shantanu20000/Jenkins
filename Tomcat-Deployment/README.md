# Install Tomcat-9
```
yum update
yum install java -y
curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.90/bin/apache-tomcat-9.0.90.tar.gz
tar -xzf apache-tomcat-9.0.90.tar.gz
cd apache-tomcat-9.0.90/
./bin/catalina.sh start
```
# Hit the Public-Ip:8080
1. click on ManageWeb follow instructions which mention in it

# Configure following file Commit line No 21 and 22

```
vim ./webapps/manager/META-INF/context.xml
```
As Like
```
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
  <!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
  allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> (commit line no 21 and 22) -->               
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```
![Screenshot (839)](https://github.com/Shantanu20000/Jenkins/assets/163661534/d4c26002-6ae7-4cc3-9599-24b7b3a684d9)

# Configure file Edit line no 56 to 61.
```
vim ./conf/tomcat-users.xml
```
As like
```
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
<!--
  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.

  Built-in Tomcat manager roles:
    - manager-gui    - allows access to the HTML GUI and the status pages
    - manager-script - allows access to the HTTP API and the status pages
    - manager-jmx    - allows access to the JMX proxy and the status pages
    - manager-status - allows access to the status pages only

  The users below are wrapped in a comment and are therefore ignored. If you
  wish to configure one or more of these users for use with the manager web
  application, do not forget to remove the <!.. ..> that surrounds them. You
  will also need to set the passwords to something appropriate.
-->
<!--
  <user username="admin" password="<must-be-changed>" roles="manager-gui"/>
  <user username="robot" password="<must-be-changed>" roles="manager-script"/>
-->
<!--
  The sample user and role entries below are intended for use with the
  examples web application. They are wrapped in a comment and thus are ignored
  when reading this file. If you wish to configure these users for use with the
  examples web application, do not forget to remove the <!.. ..> that surrounds
  them. You will also need to set the passwords to something appropriate.
-->
<!--
  <role rolename="tomcat"/>
  <role rolename="role1"/>
  <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>
  <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>
  <user username="role1" password="<must-be-changed>" roles="role1"/>
-->
          <role rolename="manager-gui"/>
          <role rolename="manager-script"/>
          <role rolename="manager-jmx"/>
          <role rolename="manager-status"/>
          <role rolename="admin-gui"/>
          <user username="linux" password="redhat" roles="manager-gui,manager-script,managare-jxm,manager-status,admin-gui"/>
</tomcat-users>
```

![Screenshot (838)](https://github.com/Shantanu20000/Jenkins/assets/163661534/e52805b1-e768-42b1-9501-82e7cf26d53f)

# Restart Catalina.sh
```
./bin/catalina.sh start
```
# Hit Public_Ip and Login with Username and Password Which mention in Above File at line no 61
# Then we Go to jenkins server and follow following step
1.  Open Jenkins Manage > Click on Credentials > Sellect "username and password" > Give our Username and Password > Give ID name > Save and Create
2. Go to Job and Configured it Syantax Creator
3. Sellect "Deploy with a container" option and fill required fields
4. Generate syntax
```
 deploy adapters: [tomcat9(credentialsId: 'linux', path: '', url: 'http://172.31.36.228:8080')], contextPath: '/', war: '**/*.war'
```
![Screenshot (838)](https://github.com/Shantanu20000/Jenkins/assets/163661534/0b0b4a6c-2166-45e5-ab22-6590b46be80b)

# In Pipeline 
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
