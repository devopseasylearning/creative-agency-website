pipeline {
  agent any
  environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}
options {
    buildDiscarder(logRotator(numToKeepStr: '20'))
    disableConcurrentBuilds()
    timeout (time: 60, unit: 'MINUTES')
    timestamps()
  }


    stages {
        stage('Setup parameters') {
            steps {
                script {
                    properties([
                        parameters([
                        
                        choice(
                            choices: ['kanibal', 'Bomber', 'Weather', 'Titan'], 
                            name: 'Application'
                                 
                                ),

                          string(name: 'WARNTIME',
                             defaultValue: '0',
                            description: '''Warning time (in minutes) before starting upgrade'''),

                          string(
                                defaultValue: 'production',
                                name: 'Please_leave_this_section_as_it_is',
                                trim: true
                            ),


                        ])
                    ])
                }
            }
        }


stage('End to End test') {

			steps {
				sh '''
                sleep 10
                '''
			}
		}


stage('SonarQube analysis') {
            agent {
                docker {
                  image 'sonarsource/sonar-scanner-cli:4.8.0'
                }
               }
               environment {
        CI = 'true'
        scannerHome='/opt/sonar-scanner'
    }
            steps{
                withSonarQubeEnv('Sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
 }


stage('Build Image kanibal_api') {
  when{   
      expression {
      env.Application == 'kanibal' }
            }

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'github-token2', variable: 'TOKEN'),
	          ]) {

	            sh '''
docker build -t devopseasylearning2021/agency:jenkins-$BUILD_NUMBER .
	            '''
	          }

	        }

	      }

	    }


stage('Build Image kanibal_consumer') {
  when{   
      expression {
      env.Application == 'kanibal' }
            }

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'github-token2', variable: 'TOKEN'),
	          ]) {

	            sh '''
                
docker build -t devopseasylearning2021/agency:jenkins-$BUILD_NUMBER .
	            '''
	          }

	        }

	      }

	    }

stage('Build Image kanibal_ui') {
  when{   
      expression {
      env.Application == 'kanibal' }
            }

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'github-token2', variable: 'TOKEN'),
	          ]) {

	            sh '''
                sleep 100
docker build -t devopseasylearning2021/agency:jenkins-$BUILD_NUMBER .
	            '''
	          }

	        }

	      }

	    }




stage('Login to dockerhub') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}



stage('Push  Image to DockerHub') {
  when{   
      expression {
      env.Application == 'kanibal' }
            }

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'github-token2', variable: 'TOKEN'),
	          ]) {

	            sh '''
docker push devopseasylearning2021/kanibal:jenkins-$BUILD_NUMBER 
	            '''
	          }

	        }

	      }

	    }
   

stage('Enviroment Test') {

			steps {
				sh '''
                sleep 10
                '''
			}
		}

stage('Server Test') {

			steps {
				sh '''
                sleep 10
                '''
			}
		}

stage('load Test') {

			steps {
				sh '''
                sleep 10
                '''
			}
		}


stage('Update kanibal Values File') {
  when{   
      expression {
      env.Application == 'kanibal' }
            }

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'github-token2', variable: 'TOKEN'),
	          ]) {

	            sh '''
                 rm -rf production-deployment || true
                 git clone https://devopseasylearning:$TOKEN@github.com/devopseasylearning/production-deployment.git 
                 cd production-deployment/kanibal
cat <<EOF > prod-values.yaml
    replicaCount: 3
    image:
      repository: devopseasylearning2021/kanibal
      pullPolicy: IfNotPresent
      tag: "jenkins-$BUILD_NUMBER"
EOF
                 
                 git config --global user.name "devopseasylearning"
                 git config --global user.email info@devopseasylearning.com
                 git add -A 
                 git commit -m "commit from Jenkins" || true
                 git push https://devopseasylearning:$TOKEN@github.com/devopseasylearning/production-deployment.git  || true
	            '''
	          }

	        }

	      }

	    }


stage('Wait for ArgoCD') {

			steps {
				sh '''
                sleep 10
                '''
			}
		}

    }


post {
    always {
      script {
        notifyUpgrade(currentBuild.currentResult, "POST")
      }
    }
    
  }

}





def notifyUpgrade(String buildResult, String whereAt) {
  if (Please_leave_this_section_as_it_is == 'origin/production') {
    channel = 'go-no-go-production'
  } else {
    channel = 'go-no-go-production'
  }
  if (buildResult == "SUCCESS") {
    switch(whereAt) {
      case 'WARNING':
        slackSend(channel: channel,
                color: "#439FE0",
                message: "Production: Upgrade starting in ${env.WARNTIME} minutes @ ${env.BUILD_URL}  Application $Application")
        break
    case 'STARTING':
      slackSend(channel: channel,
                color: "good",
                message: "Production: Starting upgrade @ ${env.BUILD_URL} Application $Application")
      break
    default:
        slackSend(channel: channel,
                color: "good",
                message: "Production: Upgrade completed successfully @ ${env.BUILD_URL}  Application $Application")
        break
    }
  } else {
    slackSend(channel: channel,
              color: "danger",
              message: "Production: Upgrade was not successful. Please investigate it immediately.  @ ${env.BUILD_URL}  Application $Application")
  }
}
