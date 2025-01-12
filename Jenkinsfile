pipeline{
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
        SCANNER_HOME = tool 'sonar-scanner'
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    stages{
        stage("Build Application"){
            	steps {
                   sh "mvn clean package"
            }

       }

       	stage("Test Application"){
          	steps {
                  sh "mvn test"
           }
       }
        stage("Sonarqube"){
            agent {
                docker {
                    image 'openjdk:17'
                }
            }
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-pass') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }   
        }
        stage("Quality Gate"){
            steps{
                script {
                    def Qualitygate = waitForQualityGate()
                    if (QualityGate.status != 'OK') {
                        error "abort pipeline: ${qualityGate.status}"
                    }
                }
            }   
        }
    }
}
