pipeline {
    agent none  // No global agent, each stage defines its own container

    environment {
        IMAGE_NAME = "manikanta513/cicd-argocd"
    }

    stages {
        stage('Build & Push Docker Image') {
            agent {
                docker {
                    image 'docker:24-dind' // or a custom image with docker CLI
                    args '-v /var/run/docker.sock:/var/run/docker.sock --user root' // bind Docker socket
                }
            }
            steps {
                script {
				   withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
						usernameVariable: 'DOCKER_USER', 
						passwordVariable: 'DOCKER_PASS')]) {
							sh '''
							docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
							echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
							docker push ${IMAGE_NAME}:${BUILD_NUMBER}
							'''
					}
                }
            }
        }
		
		stage('checkout k8s script') {
			agent {
                docker {
                    image 'bitnami/git:latest' 
					args '-u root:root'// lightweight git container
                }
            }
			steps {
				git branch: 'main',
					credentialsId: 'github-credentials',
					url: 'https://github.com/manikanta513/argocd-k8s.git'
			}
		}

        stage('Update Manifest Repo') {
            agent {
                docker {
                    image 'alpine/git' 
					args '-u root:root' // lightweight git container
                }
            }
            steps {
                script {
                    sh '''
                    sed -i "19s|.*|        image: ${IMAGE_NAME}:${BUILD_NUMBER}|" deploy.yaml
                    git config user.email "manikanta513@gmail.com"
                    git config user.name "manikanta513"
                    git add deploy.yaml
                    git commit -m "Updated deploy.yaml with image ${IMAGE_NAME}:${BUILD_NUMBER}" || echo "No changes to commit"
                    '''
                    withCredentials([usernamePassword(
                        credentialsId: 'github-credentials',
                        usernameVariable: 'GITHUB_USER',
                        passwordVariable: 'GITHUB_PASS'
                    )]) {
                        sh '''
                        git push https://${GITHUB_USER}:${GITHUB_PASS}@github.com/manikanta513/argocd-k8s.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
