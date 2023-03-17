def customImage //Variable declaration
def registry //Variable declaration

pipeline { //Start of declerative pipeline 
	agent any // Node on which pipeline should be executed
	parameters { // Parameters to be used in the pipeline
        choice(name: 'gitBranch', choices:"master", description: 'profile Name for build Maven Project')
        choice(name: 'gitCredentialsId', choices: 'github', description: 'others')
        choice(name: 'gitProjectRepo', choices: "git@github.com:technocirrus-devops/spring-boot-examples.git", description: "")
	choice(name: 'REPOSITORY_URI', choices: "639756382547.dkr.ecr.us-east-2.amazonaws.com/eureka", description: "Docker repo URL")
	choice(name: 'SERVICE_NAME', choices: 'eureka')
	choice(name: 'TASK_FAMILY', choices: 'eureka')
	choice(name: 'CLUSTER_NAME', choices: 'Eureka')
	choice(name: 'EXECUTION_ROLE_ARN', choices: 'arn:aws:iam::639756382547:role/ecsTaskExecutionRole')

    }
    environment {
	SHORT_COMMIT = "${GIT_COMMIT[0..7]}"
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
                    		def mvn_version = 'Maven' //Define the name of the maven configured in global tool configuration of Jenkins
				withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
					sh 'mvn clean install' 
				}
                	}
		}}
	}
	stage ('Sonar scan') {
        	steps {dir("spring-boot-basic-microservice/spring-boot-microservice-eureka-naming-server") {
               		withSonarQubeEnv('sonar') {
                		sh 'mvn clean package sonar:sonar'
                	}
		}
            }
        }
	stage ("Build Docker image") { //Build docker image
		steps {
			dir("spring-boot-basic-microservice/spring-boot-microservice-eureka-naming-server"){
                    		script {
					def dockerImage = docker.build("$REPOSITORY_URI", ".")
                    		}
                	}
            	}
	}
	stage ("Push docker image and clean docker images") { //Push docker image to Nexus repository
		steps {
			withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: '811ae73e-4c04-45ff-b032-e85e23a378a0']]) {
                		script {
                       			sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 639756382547.dkr.ecr.us-east-2.amazonaws.com'
                        		// Push to ECR
                        		docker.image("$REPOSITORY_URI").push("$SHORT_COMMIT")
                 
                               }
	               }
                }
	}
	stage('Deploy Image to ECS') {
        	steps{
                	withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: '811ae73e-4c04-45ff-b032-e85e23a378a0']]) {
				// prepare task definition file
                		sh """sed -e "s;%REPOSITORY_URI%;${REPOSITORY_URI};g" -e "s;%SHORT_COMMIT%;${SHORT_COMMIT};g" -e "s;%TASK_FAMILY%;${TASK_FAMILY};g" -e "s;%SERVICE_NAME%;${SERVICE_NAME};g" -e "s;%EXECUTION_ROLE_ARN%;${EXECUTION_ROLE_ARN};g" taskdef_template.json > taskdef_${SHORT_COMMIT}.json"""
                		script {
                    			// Register task definition
                    			sh ("aws ecs register-task-definition --region us-east-2  --output json --cli-input-json file://${WORKSPACE}/taskdef_${SHORT_COMMIT}.json > ${env.WORKSPACE}/temp.json")
                    			def projects = readJSON file: "${env.WORKSPACE}/temp.json"
                    			def TASK_REVISION = projects.taskDefinition.revision

                    			// update service
                    			sh ("aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --task-definition ${TASK_FAMILY}:${TASK_REVISION} --desired-count 1 --region us-east-2")
                		}
            		}
                }
	}
    }
}
