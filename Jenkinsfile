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
		stage('Deploy to ECS Fargate') {
			steps {
				script {
				def taskDef = """
					{
					\"family\": \"eureka\",
					\"containerDefinitions\": [{
						\"name\": \"eureka\",
						\"image\": \"${params.dockerrepo}:version${BUILD_NUMBER}\",
						\"cpu\": 256,
						\"memory\": 512,
						\"portMappings\": [{
						\"containerPort\": 8761,
						\"protocol\": \"tcp\"
						}]
					}]
					}
				"""
				ecsFargateRunTask taskDefinition: taskDef, cluster: 'Eureka', launchType: 'FARGATE', taskGroup: 'eureka', subnetIds: 'subnet-0418e5de3c9aee90b', securityGroupIds: 'sg-03a1603576734cfe3'
				ecsFargateUpdateService cluster: "Eureka", service: eureka, taskDefinition: "${params.dockerrepo}:version${BUILD_NUMBER}"
				}
			}
		}
	}
}



