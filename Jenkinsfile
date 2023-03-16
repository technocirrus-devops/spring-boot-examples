def customImage //Variable declaration
def registry //Variable declaration

pipeline { //Start of declerative pipeline 
	agent any // Node on which pipeline should be executed
	parameters { // Parameters to be used in the pipeline
        choice(name: 'gitBranch', choices:"master", description: 'profile Name for build Maven Project')
        choice(name: 'gitCredentialsId', choices: 'github', description: 'others')
        choice(name: 'gitProjectRepo', choices: "git@github.com:technocirrus-devops/spring-boot-examples.git", description: "")
		choice(name: 'dockerrepo', choices: "639756382547.dkr.ecr.us-east-2.amazonaws.com/eureka", description: "Docker repo URL")
    }
	stages { //block of operations to be performed under pipeline
	    stage ('Clean Workspace') { //Clean the workspace directory before other steps
          steps {
            deleteDir() //Method provided by Pipeline : Basic Steps plugin
          }
        }
		stage ("Git clone code repo"){ //Clone git repository.
				steps {
					script {
						 git branch: "${params.gitBranch}", credentialsId: "${params.gitCredentialsId}", url: "${params.gitProjectRepo}"
							sh "git checkout"	//Checkout git code
					}
				}
			}
		stage ("Maven Build") { //Maven build source code
			steps {dir("spring-boot-basic-microservice/spring-boot-microservice-eureka-naming-server") {
                script {
                    def mvn_version = 'mvn' //Define the name of the maven configured in global tool configuration of Jenkins
					withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
						sh 'mvn clean install' 
					}
                }
			}
		}

		}
		stage ("Build Docker image") { //Build docker image
			steps {
				dir("spring-boot-basic-microservice/spring-boot-microservice-eureka-naming-server"){
                    script {
                        customImage = docker.build("${params.dockerrepo}:version${BUILD_NUMBER}","-f Dockerfile .") //Using inbuilt method of docker image is built
                    }
                }
            }
		}
		stage ("Push docker image and clean docker images") { //Push docker image to Nexus repository
			steps {
				withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: '811ae73e-4c04-45ff-b032-e85e23a378a0']]) {
                script {
                       sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 639756382547.dkr.ecr.us-east-2.amazonaws.com'
			           customImage.push()
	                }
	            }
            }
        }
		stage("update the task defination") {
			steps{
				withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: '811ae73e-4c04-45ff-b032-e85e23a378a0']]) {
					script{
						NEW_ECR_IMAGE=${customImage}
						sh 'TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition eureka --region="us-east-2")'
						sh 'echo $TASK_DEFINITION | jq .containerDefinitions[0].image="${customImage}"  > task-def.json'
						sh 'aws ecs register-task-definition — family "eureka" — region="us-east-2" — cli-input-json file://task-def.json'
					}
				}
			}
		}


}
}
