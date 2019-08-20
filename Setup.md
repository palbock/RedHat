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

**Might be good to create Jenkins user and run as this user.**

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

# Rundeck

## Install
```
rpm -Uvh https://repo.rundeck.org/latest.rpm
yum install rundeck
```

### Hostname
Change hostname in _/etc/rundeck/rundeck-config.properties_.
```
grails.serverURL=http://192.168.56.2:4440
```

### Java 1.8
_This step is only necessary if Java 1.8 is not default._
Rundeck needs to run on _Java 1.8_.. Add the following in _/etc/rundeck/profile_, together with the other configured values.
```
JAVA_CMD="/usr/lib/jvm/java-1.8.0-openjdk/bin/java"
```

## Start, stop, restart and status
```
service rundeckd start
service rundeckd stop
service rundeckd restart
service rundeckd status
```

## Firewall
```
firewall-cmd --permanent --new-service=rundeck
firewall-cmd --permanent --service=rundeck --set-short="Rundeck Service Ports"
firewall-cmd --permanent --service=rundeck --set-description="Rundeck service firewalld port exceptions"
firewall-cmd --permanent --service=rundeck --add-port=4440/tcp
firewall-cmd --permanent --add-service=rundeck
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

## Change admin password
Generate MD5 hash with provided _rundeck jar_.
```
java -jar /var/lib/rundeck/bootstrap/rundeck-3.0.23-20190619.war --encryptpwd Jetty
```

Use _MD5:\<MD5 hash\>_ output and replace admin user in _/etc/rundeck/realm.properties_. 
```
admin: MD5:<MD5 hash>,user,admin,architect,deploy,build
```

## Accessing Jenkins builds
For Rundeck to access the Jenkins builds, Jenkins needs to use Matrix-based security with read-write permissions for anonymous users (or create a group for Rundeck user). After creating a job in Rundeck, add an 'Option' under 'Workflow', add `http://192.168.56.2:8080/plugin/rundeck/options/build?project=test-app&artifact=test-app.jar&limit=5&includeLastSuccessfulBuild=true&includeLastStableBuild=true` as a remoteURL. Options can then be used in scripts with the syntax: `var=@options.optionsName@`.

## Add a remote node and run a job on it
The following steps will enable nodes in a project for jobs.

* Goto: Settings (Cog wheel) -> Key Storage -> Add or Upload Key
* Create key _nodes/stage_ with SSH access to stage machine (copy the entire /root/.ssh/id_rsa)
* Goto: Projects -> New Project
* Create a project
* Goto: Project settings -> Edit nodes -> Add new node source -> URL Source
* Use this URL: _htttps://raw.githubusercontent.com/adrianhj88/redhat/master/nodes.yaml_
* Goto: Jobs -> Job Actions -> New Job
* Set name and description, and add a command. E.g. _systemctl restart test-app_
* Goto: Jobs -> <Name of new job> -> Run Job Now
 
 Steps used in workflow:  
 * `sudo lsof -i:8082 -t | xargs -r sudo kill` to kill processes on port 8082 if exists
 * `move` to execute move script
 * `download` to execute download script
 * `java -jar /nexus/artifacts/latest/test-app-*` to run latest .jar
 
 ## Trigger Rundeck job from Jenkins
 
 Use the Rundeck script in Jenkinsfile from test-app. Credentials must be added in Jenkins as well as a parameter specifying the Rundeck job ID.
 
 ## Rundeck plugin
* [Nexus3-Rundeck plugin info](https://docs.rundeck.com/plugins/nexus/2017/08/11/nexus3-options-provider.html)
* [Nexus3-Rundeck plugin github](https://github.com/nongfenqi/nexus3-rundeck-plugin)
* [Nexus3-Rundeck plugin github releases](https://github.com/nongfenqi/nexus3-rundeck-plugin/releases)

Download plugin.
```
mkdir -p /usr/local/nexus/system/com/nongfenqi/nexus/plugin/1.0.1
wget -O /usr/local/nexus/system/com/nongfenqi/nexus/plugin/1.0.1/nexus3-rundeck-plugin-1.0.1.jar https://github.com/nongfenqi/nexus3-rundeck-plugin/releases/download/1.0.1/nexus3-rundeck-plugin-1.0.1.jar
```

Update configuration files.
```
echo "bundle.mvn\:com.nongfenqi.nexus.plugin/nexus3-rundeck-plugin/1.0.1 = mvn:com.nongfenqi.nexus.plugin/nexus3-rundeck-plugin/1.0.1" >> /usr/local/nexus/etc/karaf/profile.cfg
echo "reference\:file\:com/nongfenqi/nexus/plugin/1.0.1/nexus3-rundeck-plugin-1.0.1.jar = 200" >> /usr/local/nexus/etc/karaf/startup.properties
```

Restart Nexus
```
service nexus restart
```

The plugin provides the following new HTTP resources :

- `http://192.168.56.2:8081/service/rest/rundeck/maven/options/version` : return a json array with the version of the matching artifacts.
  Parameters (all optional) :
  - `r` : repository ID to search in (null for searching in all indexed repositories)
  - `g` : groupId of the artifacts to match
  - `a` : artifactId of the artifacts to match
  - `p` : packaging of the artifacts to match ('jar', 'war', etc)
  - `c` : classifier of the artifacts to match ('sources', 'javadoc', etc)
  - `l` : limit - max number of results to return, default value is 10

- `http://192.168.56.2:8081/service/rest/rundeck/maven/options/content` : return artifact stream
  Parameters (all required) :
  - `r` : repository ID to search in (null for searching in all indexed repositories)
  - `g` : groupId of the artifacts to match
  - `a` : artifactId of the artifacts to match
  - `v` : artifact version, default value is latest version
  - `c` : classifier of the artifacts to match ('sources', 'javadoc', etc)
  - `p` : packaging of the artifacts to match ('jar', 'war', etc), default value is jar


Note that if you want to retrieve the artifact from your Rundeck script, you can use content api, example is:  
`wget "http://NEXUS_HOST/service/siesta/rundeck/maven/options/content?r=reponame&g=${option.groupId}&a=${option.artifactId}&v=${option.version}" --content-disposition`

# Add app as service
Create */etc/systemd/system/appname.service*
```
[Unit]
Description=test-app
After=syslog.target

[Service]
User=root
Group=root
ExecStart=/usr/local/intellij/test-app/target/test-app.jar

[Install]
WantedBy=multi-user.target
```

```
firewall-cmd --permanent --new-service=appname
firewall-cmd --permanent --service=appname --set-short="appname Service Ports"
firewall-cmd --permanent --service=appname --set-description="appname service firewalld port exceptions"
firewall-cmd --permanent --service=appname --add-port=4440/tcp
firewall-cmd --permanent --add-service=appname
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

# Ansible setup
[Installation guide](https://hvops.com/articles/ansible-post-install/).  

`vim /etc/ansible/ansible.cfg`
*remote_user = user*  
*remote_tmp = /tmp/*  

`vim /etc/ansible/hosts`  
*pal@192.168.56.3 ansible_user = root*  

# Bitbucket setup
[Installation guide](https://confluence.atlassian.com/bitbucketserver/install-bitbucket-server-on-linux-868976991.html). 

***Open firewall
```
firewall-cmd --permanent --new-service=bitbucket
firewall-cmd --permanent --service=bitbucket --set-short="bitbucket Service Ports"
firewall-cmd --permanent --service=bitbucket --set-description="bitbucket service firewalld port exceptions"
firewall-cmd --permanent --service=bitbucket --add-port=7990/tcp
firewall-cmd --permanent --add-service=bitbucket
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

Must connect with Jenkins through SSH keypair. Generate keypair in `/var/lib/jenkins/.ssh` and use private key in Jenkins with Bitbucket username and public key in Bitbucket repo. Repository URL is `ssh://git@192.168.56.2:7999/tes/test-app.git`.  

Use generic web hook plugin for Jenkins and Post-Receive WebHooks for Bitbucket.

