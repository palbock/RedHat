# Setup local test environment

# Jenkins installation

## Add repository and install Jenkins  
```
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install jenkins
```

## Java 8  
Sometimes, due to plugins, *Jenkins* installation needs to use Java 8. This can be set in `/etc/sysconfig/jenkins/`.

**/etc/sysconfig/jenkins**  
`JENKINS_JAVA_CMD="/usr/lib/jvm/java-1.8.0-openjdk"`

## Enable Jenkins at startup  
`chkconfig jenkins on`

## Start, stop, restart and status  
```
service jenkins start
service jenkins stop
service jenkins restart
service jenkins status
```

## Open firewall  
```
firewall-cmd --permanent --new-service=jenkins
firewall-cmd --permanent --service=jenkins --set-short="Jenkins Service Ports"
firewall-cmd --permanent --service=jenkins --set-description="Jenkins service firewalld port exceptions"
firewall-cmd --permanent --service=jenkins --add-port=8080/tcp
firewall-cmd --permanent --add-service=jenkins
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

## Configuring *Jenkins* for use with Git and Maven
Go to `192.168.56.2:8080/configureTools` and add the path to git and MAVEN_HOME to be able to fetch from remote git repo and execute maven commands via Jenkins.

# Nexus installation

## Download, extract and create user  
```
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xvzf latest-unix.tar.gz -C /usr/local/
ln -s /usr/local/nexus-3.17.0-01 /usr/local/nexus
adduser nexus
chown -R nexus:nexus /usr/local/nexus-3.17.0-01
chown -h nexus:nexus /usr/local/nexus
```

## Configure user and directories  
*Run Nexus as the "nexus"-user.*  
Uncomment and append user in `/usr/local/nexus/bin/nexus.rc`  
`run_as_user="nexus"`  

## Create directories for logging and Nexus data.  
```
install -d -o nexus -g nexus -m 750 /var/log/nexus
install -d -o nexus -g nexus -m 750 /nexus
install -d -o nexus -g nexus -m 750 /nexus/data
install -d -o nexus -g nexus -m 750 /nexus/tmp
```

Change the following jvm configurations.  
**/usr/local/nexus/bin/nexus.vomoptions**
```
/usr/local/nexus/bin/nexus.vmoptions
-XX:LogFile=/var/log/nexus/jvm.log
-Dkaraf.data=/nexus/data
-Djava.io.tmpdir=/nexus/tmp
```

## Java 8
*This step is only necessary if Java 8 is not installed or not found by Nexus. Nexus may need to run on Java 8, but should be fine 
running on Java 11.* Uncomment and use override.

**/usr/local/nexus/bin/nexus**  
`INSTALL4J_JAVA_HOME_OVERRIDE=/usr/lib/jvm/java-1.8.0-openjdk`

## Enable Nexus at startup 
```
ln -s /usr/local/nexus/bin/nexus /etc/init.d/nexus
chkconfig --add nexus
chkconfig --levels 345 nexus on
```

## Start, stop, restart and status  
```
service nexus start
service nexus stop
service nexus restart
service nexus status
```

## Open firewall  
```
firewall-cmd --permanent --new-service=nexus
firewall-cmd --permanent --service=nexus --set-short="Nexus Service Ports"
firewall-cmd --permanent --service=nexus --set-description="Nexus service firewalld port exceptions"
firewall-cmd --permanent --service=nexus --add-port=8081/tcp
firewall-cmd --permanent --add-service=nexus
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
``` 

## Gaining access to Nexus from Jenkins  
The pipeline script has an element called 'credentialsId'. Go to `192.168.56.2:8080/credentials` and create a new credential with ID 
"Nexus" and the username and password for the Nexus server.

## Downloading artifacts
The script found at [here](https://github.com/adrianhj88/test-app/blob/master/Download-from-nexus-script) can be used to download the 
latest .jar file from the Nexus repository.

Make script executable from everywhere:  
```
cd /usr/bin
ln -s locationOfFile/filename
```

*Nexus Pro* has a feature for User Tokens to replace username/password: [Nexus setup with user tokens](https://help.sonatype.com/repomanager3/security/security-setup-with-user-tokens)

# SonarQube installation

## Download, extract and set ownership  
Download *SonarQube* from [here](https://www.sonarqube.org/downloads).  
```
unzip sonarqube-7.9.zip -d /usr/local/
ln -s /usr/local/sonarqube-7.9 /usr/local/sonarqube
adduser sonarqube
chown -R sonarqube:sonarqube /usr/local/sonarqube-7.9
chown -h sonarqube:sonarqube /usr/local/sonarqube
```

Make SonarQube run as user sonarqube.  
**/usr/local/sonarqube/bin/linux-x86-64/sonar.sh**  
`RUN_AS_USER=sonarqube`

Create a *Systemd Service* for *SonarQube*.  
```
install -b -m 0644 -o root -g root /dev/stdin /etc/systemd/system/sonar.service << 'EOF'
[Unit]
Description=SonarQube
After=syslog.target network.target
 
[Service]
Type=forking
 
ExecStart=/usr/local/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/usr/local/sonarqube/bin/linux-x86-64/sonar.sh stop
 
User=sonarqube
Group=sonarqube
Restart=always
 
[Install]
WantedBy=multi-user.target
EOF
```

## System requirements  
See [SonarQube requirements](https://docs.sonarqube.org/latest/requirements/requirements). 

## Enable SonarQube at startup  
`systemctl enable sonar`

## Start, stop, restart and status  
```
systemctl start sonar
systemctl stop sonar
systemctl restart sonar
systemctl status sonar
``` 

## Open firewall  
```
firewall-cmd --permanent --new-service=sonarqube
firewall-cmd --permanent --service=sonarqube --set-short="SonarQube Service Ports"
firewall-cmd --permanent --service=sonarqube --set-description="SonarQube service firewalld port exceptions"
firewall-cmd --permanent --service=sonarqube --add-port=9000/tcp
firewall-cmd --permanent --add-service=sonarqube
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

SonarQube will run on `192.168.56.2:9000`. Create a project and use the token created there as a "secret text" credential in *Jenkins* 
with ID "SonarQube". *SonarQube Scanner* must be added in "Manage Jenkins" -> "Global Tool Configuration". Name it "SonarQubeScanner" 
and select "Install automatically". Set up *SonarQube* server under "Manage Jenkins" → "Configure System". Name it *SonarQube*, set 
URL to `http://192.168.56.2:9000` and add the authentication token created earlier.

# Jenkins plugins

Install the following list of plugins, and all the recommended plugins, for Jenkins.

* Nexus Artifact Uploader
*Nexus Platform Plugin
* Pipeline
* Pipeline Utility Steps
* Publish Over SSH
* SonarQube Scanner for Jenkins
* SSH Plugin

# Jenkins pipeline

[This](https://github.com/adrianhj88/test-app/) github repository has an example of [Jenkinsfile](https://github.com/adrianhj88/test-app/blob/master/Jenkinsfile) and [sonar-project.properties](https://github.com/adrianhj88/test-app/blob/master/sonar-project.properties) set up to work with the previous installation 
instructions.

## Pipeline problems
* The *Jenkins* pipeline is missing the step of deploying the build to a runtime environment. The SSH and *Nexus* plugins could be used for this solution.
* The *waitForQualityGate* part of the SonarQube stage does not work well without a sleep first. Could be caused by webhooks that is not working properly.
