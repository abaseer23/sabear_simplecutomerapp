pipeline {
    agent any
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "maven"
        
    }
	 environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "52.90.95.78:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "hello"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus"
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/abaseer23/sabear_simplecutomerapp.git';
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    sh 'mvn -Dmaven.test.failure.ignore=true clean install'
                }
            }
        }
        stage('SonarCloud') {
            steps {
                withSonarQubeEnv('sonarqube_server') {
                    sh '$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Ncodeit \
                    -Dsonar.projectName=Ncodeit \
                    -Dsonar.projectVersion=2.0 \
                    -Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/ \
                    -Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports \
                    -Dsonar.jacoco.reportPath=target/jacoco.exec \
                    -Dsonar.java.binaries=src/com/room/sample'
                }
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM file using XMLSlurper
                    def pomContent = readFile("pom.xml")
                    def pom = new XmlSlurper().parseText(pomContent)
                    
                    // Get packaging type (default to 'jar' if not specified)
                    def packaging = pom.packaging?.text() ?: 'jar'
                    
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${packaging}");
                    
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId.text()}, packaging: ${packaging}, version ${pom.version.text()}";
                        
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId.text(),
                            version: pom.version.text(),
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId.text(),
                                classifier: '',
                                file: artifactPath,
                                type: packaging],
                                // Lets upload the pom.xml file for additional information
                                [artifactId: pom.artifactId.text(),
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
    }
}
