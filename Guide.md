# VB Setup

* Download and install VirtualBox and extension pack from https://www.virtualbox.org/wiki/Downloads  
* Create user on redhat.com and download RHEL 7.5 image from https://developers.redhat.com/products/rhel/download.  
* Create 2 virtual machines  (build and stage) on VB using this image. Install as basic web server with 2048 MB RAM and 20 GB Disk.
Creating root user is optional.  
* For both VMs, enable 2 adapters: NAT + Host Only  
* Under file -> host network manager, turn off DHCP Server.  

# RHEL setup

### Start the VM and type in command line:

### vim /etc/sysconfig/network-scripts/ifcfg-enp0s3:  
Add `""` around the variables, change `ONBOOT` to `"yes"` and add `ZONE=public`.  

### vim /etc/sysconfig/network-scripts/ifcfg-enp0s8:
Should look like this:
```
DEVICE="enp0s8"  
BOOTPROTO="static"  
HWADDR (optional)  
NM_CONTROLLER="yes"  
ONBOOT="yes"  
TYPE="Ethernet"  
IPADDR="your_desired_ip_address (192.168.56.2)"  
NETMASK="255.255.255.0"  
ZONE=public  
```
Run `sudo service network restart`

Check if correct with `ifconfig` and `ping vg.no`.

Run the following commands:
```
subscription manager register --username "username" --password "password" --auto-attach (username and password for redhat user)

sudo yum install openssh-server

sudo yum install java-1.8.0-openjdk-devel

sudo yum install git
```

If any installs fail, run `sudo yum update` first.

# Ubuntu setup

* Install Linux shell for windows using this guide: https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/

Run
```
mkdir .ssh  
vim .ssh/config  
```
### Add users in config file:  
Host build  
  &nbsp;&nbsp;Hostname 192.168.25.2  
  &nbsp;&nbsp;User pal  
  
Run
```
ssh-keygen  
ssh build  
```
If problem with ownership: chown 600 .ssh/config or chmod 600 .ssh/config  
Run `ssh-copy-id` build after logging in with password to not have to enter password each time if you have created an admin user.  

# Jenkins

### Install
```
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install jenkins
```

### Start/Stop
```
service jenkins start/stop/restart
chkconfig jenkins on
```

### Firewall
```
firewall-cmd --permanent --new-service=jenkins
firewall-cmd --permanent --service=jenkins --set-short="Jenkins Service Ports"
firewall-cmd --permanent --service=jenkins --set-description="Jenkins service firewalld port exceptions"
firewall-cmd --permanent --service=jenkins --add-port=8080/tcp
firewall-cmd --permanent --add-service=jenkins
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

### Git

It may be necessary to add the path to the git executable to Jenkins configuration. Go to "Global tool configuration", "Git" and add the path. Should be `/usr/bin/git` by default.

### Running JUnit tests on Jenkins

* Install Maven on Linux VM: https://tecadmin.net/install-apache-maven-on-centos/  
* Go to configure tab on your project in Jenkins.  
* Add build step: "Invoke top-level maven targets"
* Enter "clean test"  

* Add post-build step: "Publish JUnit test result report".   
* Enter `target/surefire-reports/*.xml`.

# Nexus

### Install
Download and extract _Nexus_. Create and set a _nexus_-user as owner for the directories.
```
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xvzf nexus-3.17.0-01-unix.tar.gz -C /usr/local/
ln -s /usr/local/nexus-3.17.0-01 /usr/local/nexus
adduser nexus
chown -R nexus:nexus /usr/local/nexus-3.17.0-01
chown -h nexus:nexus /usr/local/nexus
```

Run _Nexus_ as the created _nexus_-user. Uncomment and append user in _bin/nexus.rc_.
```
run_as_user="nexus"
```

Change to the following configurations in _bin/nexus.vmoptions_.
```
-XX:LogFile=/var/log/nexus/jvm.log
-Dkaraf.data=/nexus/data
-Djava.io.tmpdir=/nexus/tmp
```

Create directories to match configurations from _bin/nexus.vmoptions_.
```
install -d -o nexus -g nexus -m 750 /var/log/nexus
install -d -o nexus -g nexus -m 750 /nexus
install -d -o nexus -g nexus -m 750 /nexus/data
install -d -o nexus -g nexus -m 750 /nexus/tmp
```

### Java 1.8
_This step is only necessary if Java 1.8 is not installed or not found by Nexus._
Nexus needs to run on _Java 1.8_.. Uncomment and use override in _bin/nexus_.
```
INSTALL4J_JAVA_HOME_OVERRIDE=/usr/lib/jvm/java-1.8.0-openjdk
```

### Start/Stop
```
ln -s /usr/local/nexus/bin/nexus /etc/init.d/nexus
chkconfig --add nexus
chkconfig --levels 345 nexus on
service nexus start
```

### Firewall
```
firewall-cmd --permanent --new-service=nexus
firewall-cmd --permanent --service=nexus --set-short="Nexus Service Ports"
firewall-cmd --permanent --service=nexus --set-description="Nexus service firewalld port exceptions"
firewall-cmd --permanent --service=nexus --add-port=8081/tcp
firewall-cmd --permanent --add-service=nexus
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

## Alternativ måte

https://help.sonatype.com/learning/repository-manager-3/first-time-installation-and-setup/lesson-1%3A--installing-and-starting-nexus-repository-manager
