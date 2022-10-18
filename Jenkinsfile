pipeline {
    agent any
    tools{
        maven 'maven'
      }
    environment{
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "127.0.0.1:8081"
        NEXUS_CREDENTIAL_ID = "nexus"
    }  

    stages{

        stage('Build'){
            steps{
           
                      sh 'mvn clean package'
               
            }
         }

        stage('Sonarqube'){
            environment{
                scannerHome = tool 'sonarscanner'
            }    
            steps{
                withSonarQubeEnv('sonarqube'){
                    sh "mvn sonar:sonar -Dsonar.projectKey=projet-ci "
                }         
            }
        }
        
        stage("publish  war file on nexus"){
            steps{
               script{
                    
                   pom = readMavenPom file: "pom.xml";                   
                    filesByGlob = findFiles(glob: "target/*.war");
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                            
                    if(artifactExists){
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: "WebApp",
                            credentialsId: NEXUS_CREDENTIAL_ID,     
                            artifacts:[   
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: 'war'],
                            ]
                        );
                    }else{
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }             
    }
        /*  post {
          always 
          {
            script 
            {
            def url = "${env.BUILD_URL}"
            def status = currentBuild.currentResult
            def color = status == 'SUCCESS' ? '#00FF00' : '#FF0000'
            def resultIcon = status == 'SUCCESS' ? ':white_check_mark:' : ':anguished:'
            slackSend (message: "${resultIcon} Jenkins Build $currentBuild.currentResult\n\nResults available at: [ Jenkins-$env.JOB_NAME#$env.BUILD_NUMBER ] \n ${url}", 
                        color: color)
            }
          }
      }*/
}
