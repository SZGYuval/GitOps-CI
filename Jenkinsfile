pipeline {
    agent any

    environment {
      NAME = "python-app"
      VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
      IMAGE_REPO = "szgyuval123/gitops-repo"
      GIT_TOKEN = credentials('github-creds')
    }

    stages {
        stage('Build Image') {
            steps {
                sh "docker build -t ${NAME} ."
                sh "docker tag ${NAME}:latest ${IMAGE_REPO}:${NAME}-${VERSION}"
            }
        }

        stage('Tests') {
            steps {
                echo "Tests should be added!"
            }
        }

        stage('Push Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-hub-creds', url: '') {
                    sh  "docker image push ${IMAGE_REPO}:${NAME}-${VERSION}"
                }
            }
        }

        stage('Clone/Pull Repo') {
            steps {
                script {
                    if (fileExists('GitOps-CICD')) {
                        echo 'Cloned repo already exists - Pulling latest changes'

                        dir("GitOps-CICD") {
                            sh 'git pull https://github.com/SZGYuval/GitOps-CICD'
                        }
                    }
                    else {
                        echo 'Repo does not exists - Cloning the repo'
                        sh 'git clone https://github.com/SZGYuval/GitOps-CICD'
                    }
                }
            }
        }

        stage('Update manifest') {
            steps {
                dir("GitOps-CICD") {
                    sh 'sed -i "s|szgyuval123.*|szgyuval123/gitops-repo:python-app-8-006154380cc9f148996d0e7424a097e581c6798f|" deployment.yaml'
                }
            }
        }
    }
}