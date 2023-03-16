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
						//sh 'docker run --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven:3.3-jdk-8 mvn clean install'
						sh 'mvn clean install' // -DskipTests=true -B' //Maven build
						#sh "mvn checkstyle:checkstyle"
						#recordIssues(tools: [checkStyle(reportEncoding: 'UTF-8')])
						sh "cp */target/*.jar Docker/eureka/" //Copy jar files from target directory to Docker directory
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
					//docker.withRegistry(url: '${registry}', credentialsId: 'e874664f-6680-4efa-bccd-c0dd15626491') { //Using inbuilt method withRegistry we can interact with custom registires
					//withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'e874664f-6680-4efa-bccd-c0dd15626491', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
	sh "docker login --password=${PASSWORD} --username=${USERNAME} ${registry}"

					customImage.push() //Push the docker image
					}
                   			 sh "docker rmi ${params.dockerrepo}/eureka-repo:version${BUILD_NUMBER}" //Remove the local docker image
               			 }
           		 }
		}
		stage ("Start containers") { //Start docker containers
			steps {
				dir("Docker/"){ //Execute script in Docker directory
					script {
						//docker.withRegistry(${registry}, 'e874664f-6680-4efa-bccd-c0dd15626491') { //Using inbuilt method withRegistry to pull docker image
						 withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'e874664f-6680-4efa-bccd-c0dd15626491', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
        						//sh "docker login --password=${PASSWORD} --username=${USERNAME} ${registry}"
							//sh "sed -i \"s;latest;version${BUILD_NUMBER};g\" docker-compose.yml" //Modifying yml file with the tag name for new docker image
							//sh "sed -i \"s;reponame;${params.dockerrepo};g\" docker-compose.yml"
							//sh "docker-compose up -d" //Starting docker container 
							sh "sed -i \"s;latest;version${BUILD_NUMBER};g\" springBootMongo.yml" //Modifying yml file with the tag name for new docker image
							sh "sed -i \"s;reponame;${params.dockerrepo};g\" springBootMongo.yml"
							def ipaddress = sh(script: "cat ../ipaddress.txt", returnStdout: true)
							//sh "sed -i \"s/ipaddr/${ipaddress}/g\" script.sh"
							sh "chmod +x script.sh"
							sh "sh script.sh"
								}
					}
				}
			}
		
		}
	}
	post {
      		always {
        		junit '*/target/surefire-reports/*.xml'
      		}
   	} 
}
