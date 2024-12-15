pipeline {

        agent any

	environment {
                SONAR_SCANNER_HOME = tool 'SonarQube Scanner'
		IMAGE_TAG = 'latest'
                ECR_REPO = 'greenpiece-repo'
                ECR_REGISTRY = '381492139836.dkr.ecr.us-west-2.amazonaws.com'
                //ECS_CLUSTER = 'greenpiece-cluster'
                //ECS_SERVICE = 'greenpiece-service'
                //ECS_TASK_DEFINITION = 'mediplus-taskdef'
                TRIVY_IMAGE = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
	}
	
	stages {

                stage('Checkout Git') {
                        steps {
                        	git branch: 'main', changelog: false, poll: false, url: 'https://github.com/max-az-10/greenpiece.git'
			}
                }

		stage('SonarQube Analysis') {
			steps {	
				withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'Sonar_Token')]) {
   					withSonarQubeEnv('SonarQube') {
						sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner"
    					}	
  				}
			}
		}
		stage('Build & Tag image') {
                        steps {
				withCredentials([usernamePassword(credentialsId: 'Aws-cred', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
					script {
                				sh "docker build -t ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG} ."
				        }
                		}
			}
		}
		stage('Trivy scan') {
                        steps {
				withCredentials([usernamePassword(credentialsId: 'Aws-cred', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                                        script {
                                                sh "trivy image --severity HIGH,MEDIUM --format table -o trivy-report.html ${TRIVY_IMAGE}"
                                        }
                                }        	       	

                        }
                }
		stage('Login & Push to ECR') {
                        steps {
                		withCredentials([usernamePassword(credentialsId: 'Aws-cred', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                                        script {
                                                sh """
						aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin ${ECR_REGISTRY}
						docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}							
						"""
                                        }
                                }
					
                        }
                }
		stage('Update services') {
                        steps {
                                withCredentials([usernamePassword(credentialsId: 'Aws-cred', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                                        script {
                                        	sh "aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --task-definition $ECS_TASK_DEFINITION  --force-new-deployment"
                                        }
                                }

                        }
                }

	}
}
