def component = [
		Preprocess: false,
		Hyper: false,
		Train: false,
		Test: false,
		Bento: false
]
pipeline {
	agent any
	stages {
		stage("Checkout") {
			steps {
				checkout scm
			}
		}
		stage("Build") {
			steps {
				script {
					component.each{ entry ->
						stage ("${entry.key} Build"){
							if(entry.value){
								var = entry.key
								sh "docker-compose build ${var.toLowerCase()}"
							}
						}
					}
				}
			}
		}
		stage("Tag and Push") {
			steps {
				script {
					component.each{ entry ->
						stage ("${entry.key} Push"){
							if(entry.value){
								var = entry.key
								withCredentials([[$class: 'UsernamePasswordMultiBinding',
								credentialsId: 'docker_credentials',
								usernameVariable: 'DOCKER_USER_ID',
								passwordVariable: 'DOCKER_USER_PASSWORD'
								]]){
								sh "docker tag sp_${var.toLowerCase()}:latest ${DOCKER_USER_ID}/sp_${var.toLowerCase()}:${BUILD_NUMBER}"
								sh "docker login -u ${DOCKER_USER_ID} -p ${DOCKER_USER_PASSWORD}"
								sh "docker push ${DOCKER_USER_ID}/sp_${var.toLowerCase()}:${BUILD_NUMBER}"
								}
							}
						}
					}
				}
			}	
		}
		stage("publish") {
			steps {
				script {
					sshPublisher(publishers: [sshPublisherDesc(configName: 'ubuntu', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd deploy
docker-compose up -d''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/deploy', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '/*')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
				}
			}
		}
	}
}