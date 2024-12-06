pipeline {
    agent any
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "MVN_HOME"
        
    }
	 environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "44.211.152.131:8081/"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "devops"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus_server"
  
        SLACK_CHANNEL = "jenkins-integration"  // Example Slack channel
        SLACK_COLOR_SUCCESS = "good"           // Green color for success
        SLACK_COLOR_FAILURE = "danger"         // Red color for failure
        SLACK_CREDENTIAL_ID = "secret"
        TOMCAT_CREDENTIALS_ID="tomcat_credentials"
        TOMCAT_URL="http://107.22.59.24:8080"
        APP_NAME = "SimpleCustomerApp-7-SNAPSHOT"
        WAR_FILE = "/var/lib/jenkins/workspace/declarative-job/target/SimpleCustomerApp-7-SNAPSHOT.war"
    }		
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/kishangujja/sabear_simplecutomerapp.git';
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh 'mvn -Dmaven.test.failure.ignore=true install'
                }
            }
        }
        stage("publish to nexus") {
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
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
			    groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    
	stage("Deploy War file to Tommcat"){
            steps{
               sshagent(['tomcat-credentials']) {
    sh """
        # Copy WAR file to Tomcat webapps directory
        scp -o StrictHostKeyChecking=no target/*.war ec2-user@107.22.59.24:/opt/apache-tomcat-9.0.97/webapps/

        # Stop Tomcat server
        ssh -o StrictHostKeyChecking=no ec2-user@107.22.59.24 'sudo /opt/apache-tomcat-9.0.97/bin/shutdown.sh'

        # Start Tomcat server
        ssh -o StrictHostKeyChecking=no ec2-user@107.22.59.24 'sudo /opt/apache-tomcat-9.0.97/bin/startup.sh'
    """
}

            }
        }
    }
	post {
        success {
            slackSend(
                channel: env.SLACK_CHANNEL, 
                message: "Declarative pipeline Build #${BUILD_NUMBER} succeeded! by Gujja Kishan Rao :white_check_mark:", 
                color: env.SLACK_COLOR_SUCCESS
            )
        }
        failure {
            slackSend(
                channel: env.SLACK_CHANNEL, 
                message: "Build #${BUILD_NUMBER} failed! :x:", 
                color: env.SLACK_COLOR_FAILURE
            )
        }
    }
}

