# Installation of Maven

```
    apt install git -y
    curl -O https://dlcdn.apache.org/maven/maven-3/3.9.7/binaries/apache-maven-3.9.7-bin.zip
    unzip apache-maven-3.9.7-bin.zip
    mv apache-maven-3.9.7 /opt/
    apt install maven -y
    mvn --version
    rm apache-maven-3.9.7-bin.zip 
    apt search java
    sudo apt update
    sudo apt install openjdk-11-jdk
    sudo update-alternatives --config java
    sudo vim /etc/profile.d/maven.sh 
		export M2_HOME=/opt/maven
		export PATH=$M2_HOME/bin:$PATH   
    source /etc/profile.d/maven.sh
```
# Package of Code
```
/opt/maven/bin/mvn clean package
```
