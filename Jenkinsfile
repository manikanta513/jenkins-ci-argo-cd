 pipeline {
    
    agent {
        docker {
            image 'manikanta513/jenkins-agent:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock --user root'
            reuseNode true
        }
    }
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps { 
                url: 'https://github.com/manikanta513/Jenkins-Zero-To-Hero',
                branch: 'main'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    dir('python-jenkins-argocd-k8s'){
                        sh '''
                        echo 'Buid Docker Image'
                        docker build -t manikanta513/cicd-argocd:${BUILD_NUMBER} .
                        '''
                    }
                }
            }
        }

        stage('Push the artifacts'){
           steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                 usernameVariable: 'DOCKER_USER', 
                                 passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                        echo 'docker login'
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        echo 'Push to Repo'
                        docker push manikanta513/cicd-argocd:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        
        stage('Update K8S manifest'){
            steps {
                script{
                    dir('python-jenkins-argocd-k8s/deploy'){
                        withCredentials([usernamePassword(credentialsId: 'github-credentials', 
                                 usernameVariable: 'GITHUB_USER', 
                                 passwordVariable: 'GITHUB_PASS')]) {
                            sh """
                            cat deploy.yaml
                            sed -i "19s|.*|        image: manikanta513/cicd-argocd:${BUILD_NUMBER}|" deploy.yaml
                            cat deploy.yaml
                            git config user.email "manikanta513@gmail.com"
                            git config user.name "manikanta513"
                            git add deploy.yaml
                            git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                            git remote -v
                            git push https://${GITHUB_USER}:${GITHUB_PASS}@github.com/manikanta513/Jenkins-Zero-To-Hero.git HEAD:main
                            """                       
                    }
                }
            }
        }
    }
}
}
