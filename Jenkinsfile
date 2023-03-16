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
				script {
					registry = "http://${params.dockerrepo}"
					print "Resitry URL is : ${registry}"
					docker.withRegistry(url: 'https://639756382547.dkr.ecr.us-east-2.amazonaws.com') { //Using inbuilt method withRegistry we can interact with custom registires
					//withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'e874664f-6680-4efa-bccd-c0dd15626491', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
					customImage.push() //Push the docker image
					}
                   			 sh "docker rmi ${params.dockerrepo}:version${BUILD_NUMBER}" //Remove the local docker image
               			 }
           		 }
	}
}
}
