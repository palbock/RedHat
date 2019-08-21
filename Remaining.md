## Update credentials in Nexus  
Currently the script for downloading latest artifact uses username and password in plain text. Nexus Pro has an option to use tokens.

## Integrate Rundeck with Nexus  
Currently the rollback job in Rundeck selects artifacts from Jenkins. It would be preferable to use Nexus instead as Nexus is a more
reliable repository. The JSON file for Nexus artifacts can be accessed from the browser when logged into Nexus, but not from a Rundeck
option (http://192.168.56.2:8081/service/rest/rundeck/maven/options/version?r=test-app&g=no.ahj&a=test-app&p=jar).
