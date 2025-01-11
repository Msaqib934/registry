pipeline {
    agent { label 'Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "msaqib934"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
	}
	stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Msaqib934/registry-app.git'
                }
        }
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
	stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }

       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }
	stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
       }
	stage("Checkout from SCM for argocd") {
               steps {
                   git branch: 'main', credentialsId: 'github', url: 'https://github.com/Msaqib934/argocd'
               }
        }
	stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "argocd"
            GIT_USER_NAME = "Msaqib934"
	    APP_NAME = "register-app-pipeline"
        }
        steps {
            withCredentials([string(credentialsId: 'github2', variable: 'GITHUB_TOKEN')]) {
                sh """
                    git config user.email "msaqib934@gmail.com"
                    git config user.name "Msaqib934"
                    IMAGE_TAG=${IMAGE_TAG}
                    sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' dev/deployment.yaml
                    git add dev/deployment.yaml
                    git commit -m "Update deployment image to version ${IMAGE_TAG}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                """
            }
        }
    }
  }
}
