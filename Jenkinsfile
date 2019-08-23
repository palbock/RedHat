pipeline {
    agent any

    environment {
        RUNDECK_PW = credentials('rundeck-pw')
    }

    stages {
        stage('Build') {
            steps {
                /*script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    currentBuild.displayName = "${filesByGlob[0].name} ${filesByGlob[0].lastModified}"
                }*/
                sh 'mvn clean install'
            }
        }
        /*stage('Unit tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration tests') {
            steps {
                echo 'Integration tests..'
            }
        }
        stage('SonarQube') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }

            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }

                sleep(60)

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploying to Nexus') {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: "nexus3",
                            protocol: "http",
                            nexusUrl: "192.168.56.2:8081",
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: "test-app",
                            credentialsId: "Nexus",
                            artifacts: [
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ],
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging]
                                ]
                          );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Archiving') {
            steps {
                echo "Archiving artifact file: ${artifactPath}-${pom.groupId}.${pom.packaging}-${pom.version}";

                archiveArtifacts artifacts: 'target/test-app.jar',
                onlyIfSuccessful: true
            }
        }*/
        /*stage('Email') {
            steps{
                emailext(
                body: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful",
                //recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], WILL SEND EMAIL TO PERSON CHECKING IN CHANGES
                to: '$DEFAULT_RECIPIENTS',
                subject: 'Jenkin build'
                )
            }
        }*/
        stage('Rundeck') {
            steps {
                step(
                    [
                        $class: 'RundeckNotifier',
                        includeRundeckLogs: true,
                        jobId: "${params.RUNDECK_JOB_ID}",
                        jobPassword: '$RUNDECK_PW',
                        jobUser: 'admin',
                        notifyOnAllStatus: false,
                        options: 'job-options',
                        rundeckInstance: 'Rundeck',
                        shouldFailTheBuild: true,
                        shouldWaitForRundeckJob: true,
                        tailLog: false
                    ]
                )
            }
        }
    }
    post {
             always {
                 echo 'This will always run'
             }
             success {
                 echo 'This will run only if successful'
             }
             failure {
                 emailext(
                       body: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful",
                       //recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], WILL SEND EMAIL TO PERSON CHECKING IN CHANGES
                       to: '$DEFAULT_RECIPIENTS',
                       subject: 'Jenkins build'
                       )
             }
             unstable {
                 echo 'This will run only if the run was marked as unstable'
             }
             changed {
                 echo 'This will run only if the state of the Pipeline has changed'
                 echo 'For example, if the Pipeline was previously failing but is now successful'
             }
         }
}
