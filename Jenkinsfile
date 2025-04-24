pipeline{
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
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
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'sonar') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }   
        }
        stage("Quality Gate"){
            steps{
                script {
                    def qualitygate = waitForQualityGate()
                    if (qualitygate.status != 'OK') {
                        error "abort pipeline: ${qualitygate.status}"
                    }
                }
            }   
        }
        stage("Push Code to Nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nex', variable: 'nexus')]) {
                        sh '''
                            docker build -t 18.212.246.146:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus 18.212.246.146:8083
                            docker push 18.212.246.146:8083/springapp:${VERSION}
                            docker rmi 18.212.246.146:8083/springapp:${VERSION}
                        '''
                    }  
                }
            }   
        }
        stage("Deploy to Kubernetes") {
            steps {
                script {
                    dir('dev/') {
                                sh '''
                                    ctr image pull --hosts-dir "/etc/containerd/certs.d" 18.212.246.146:8083/springapp:${VERSION}
                                    kubectl apply -f deployment.yaml
                                    kubectl set image deployment/myapp myapp=18.212.246.146:8083/springapp:${VERSION} -n default
                                    kubectl apply -f service.yaml
                                '''
                    }
                }
            }
        }
    }
}
