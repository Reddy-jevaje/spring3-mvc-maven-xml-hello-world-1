pipeline {
    agent any 
    tools {
        // Note: this should match with the tool name configured in your Jenkins instance (JENKINS_URL/configureTools/)
        maven "MVN_HOME"
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "3.80.48.88:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "devops"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-server"
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/Reddy-jevaje/spring3-mvc-maven-xml-hello-world-1.git'
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
                    // Ensure the Pipeline Utility Steps plugin is installed for readMavenPom to work
                    try {
                        // Read POM xml file using 'readMavenPom' step
                        def pom = readMavenPom file: 'pom.xml'
                        
                        // Find built artifact under target folder
                        def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                        
                        // Print some info from the artifact found
                        echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                        
                        // Extract the path from the File found
                        def artifactPath = filesByGlob[0].path
                        
                        // Check if the artifact exists
                        def artifactExists = fileExists artifactPath
                        
                        if (artifactExists) {
                            echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version: ${BUILD_NUMBER}"
                            
                            // Upload the artifact to Nexus
                            nexusArtifactUploader(
                                nexusVersion: NEXUS_VERSION,
                                protocol: NEXUS_PROTOCOL,
                                nexusUrl: NEXUS_URL,
                                groupId: pom.groupId,
                                version: "${BUILD_NUMBER}",
                                repository: NEXUS_REPOSITORY,
                                credentialsId: NEXUS_CREDENTIAL_ID,
                                artifacts: [
                                    [artifactId: pom.artifactId,
                                     classifier: '',
                                     file: artifactPath,
                                     type: pom.packaging],
                                    [artifactId: pom.artifactId,
                                     classifier: '',
                                     file: 'pom.xml',
                                     type: 'pom']
                                ]
                            )
                        } else {
                            error "*** File: ${artifactPath}, could not be found"
                        }
                    } catch (Exception e) {
                        error "Error reading the POM file: ${e.message}"
                    }
                }
            }
        }
    }
}
