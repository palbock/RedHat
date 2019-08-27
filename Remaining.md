## Update credentials in Nexus  
Currently the script for downloading latest artifact uses username and password in plain text. Nexus Pro has an option to use tokens.

## Update credentials in Rundeck
Currently Jenkins stores Rundeck credentials as username/password. Would be preferable to use secret key or similar. Investigate if possible.

Credential 'rundeck-pw' can be stored as an API token generated in Rundeck.

## Integrate Rundeck with Nexus  
Currently the rollback job in Rundeck selects artifacts from Jenkins. It would be preferable to use Nexus instead as Nexus is a more
reliable repository. The JSON file for Nexus artifacts can be accessed from the browser when logged into Nexus, but not from a Rundeck
option (http://192.168.56.2:8081/service/rest/rundeck/maven/options/version?r=test-app&g=no.ahj&a=test-app&p=jar).

`curl -X GET "http://192.168.56.2:8081/service/rest/v1/search/assets?repository=test-app&maven.extension=jar" -H "accept: application/json"` gives a list of jars with downloadURL. Possible to combine with Rundeck somehow?

## Notifications
Setup email (or other) notification for when build is run/successful/failed.
