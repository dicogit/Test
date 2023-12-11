pipeline {
	agent any 
	environment {
		IMAGE_PHP="devopsdr/pub:php$BUILD_NUMBER"
		IMAGE_DB="devopsdr/pub:DB$BUILD_NUMBER"
		BUILD_IP='ec2-user@3.111.51.68'
		DEPLOY_IP='ec2-user@13.233.26.115'
	}
	stages {
		stage ('BUILD THE PHPDB IMAGE') {
			steps {
				sshagent(['ssh-agent']) {
					script {
						withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dpwd', usernameVariable: 'docr')]) {
							echo "BUILD THE PHPDB IMAGE"
							sh "scp -o StrictHostKeyChecking=no -r php_files db_files ${BUILD_IP}:/home/ec2-user/"
							sh "ssh -o StrictHostKeyChecking=no ${BUILD_IP} 'bash ~/php_files/php_script.sh'"
							sh "ssh -o StrictHostKeyChecking=no ${BUILD_IP} 'sudo docker build . -t ${IMAGE_PHP} -f /home/ec2-user/php_files/dockerfile'"
							sh "ssh -o StrictHostKeyChecking=no ${BUILD_IP} 'sudo docker build . -t ${IMAGE_DB} -f /home/ec2-user/db_files/dockerfile'"
							sh "ssh -o StrictHostKeyChecking=no ${BUILD_IP} 'sudo docker login -u ${docr} -p ${dpwd}'"
							sh "ssh -o StrictHostKeyChecking=no ${BUILD_IP} 'sudo docker push ${IMAGE_PHP}'"
							sh "ssh -o StrictHostKeyChecking=no ${BUILD_IP} 'sudo docker push ${IMAGE_DB}'"
						}
					}
				}
			}
		}
		stage ('DEPLOY THE PHPDB CONTAINER') {
			steps {
				sshagent(['ssh-agent']) {
					script {
						withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dpwd', usernameVariable: 'docr')]) {
							echo "DEPLOY THE PHPDB CONTAINER"
							sh "scp -o StrictHostKeyChecking=no -r compose_phpdb ${DEPLOY_IP}:/home/ec2-user/"
							sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_IP} 'bash ~/compose_phpdb/docker-compose-script.sh'"
							sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_IP} 'sudo docker login -u ${docr} -p {dpwd}'"
							sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_IP} 'sudo php=${IMAGE_PHP} db=${IMAGE_DB} docker-compose -f /home/ec2-user/compose_phpdb/docker-compose.yml up -d'"
						}
					}
				}
			}
		}
	}
}  
	