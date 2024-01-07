pipeline{
    agent{
        label "jenkins-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "complete-cicd-project"
        RELEASE = "1.0.0"
        DOCKER_USER = "aecloud"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'develop', credentialsId: 'github', url: 'https://github.com/Abdelhadi-ezziat/complete_cicd_project'
            }

        }

        stage("Build Application"){
            steps {
                sh "mvn clean install"
            }
        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }
        }

        stage("Sonarqube Analysis"){
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
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

        stage("Build and push Docker image"){
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build(IMAGE_NAME)
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trigger CD pipeline"){
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins-dev.com/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
                }
            }
        }

        // }

        // stage("Trivy Scan") {
        //     steps {
        //         script {
		//    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image dmancloud/complete-prodcution-e2e-pipeline:1.0.0-22 --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
        //         }
        //     }

        // }

        // stage ('Cleanup Artifacts') {
        //     steps {
        //         script {
        //             sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
        //             sh "docker rmi ${IMAGE_NAME}:latest"
        //         }
        //     }
        // }



    }

    // post {
    //     failure {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
    //                 mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
    //         }
    //      success {
    //            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
    //                 mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
    //       }      
    // }
}
