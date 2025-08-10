pipeline {
    agent any

    tools {
        git 'Default'
    }

    environment {
      NAME = "python-app"
      VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
      IMAGE_REPO = "szgyuval123/gitops-repo"
      GIT_TOKEN = credentials('github-creds')
      PROJECT_NAME = "${JOB_NAME}"
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

        stage('Creating Secret - CI stage') {
            steps {
                sh 'argocd-vault-plugin generate -c /root/vault.env - < secret.yaml | kubectl apply -f -'
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
                    sh 'sed -i "s|szgyuval123.*|${IMAGE_REPO}:${NAME}-${VERSION}|" deployment.yaml'
                }
            }
        }

        stage('Giving jenkins permissions') {
            steps {
                sh "sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/${PROJECT_NAME}/GitOps-CICD"
            }
        }

        stage('Pushing Changes') {
            steps {
                dir("GitOps-CICD") {
                    sh "git config --global user.name Jenkins Bot"
                    sh "git config --global user.email jenkins@exmaple.com"
                    sh "git remote set-url origin https://${GIT_TOKEN}@github.com/SZGYuval/GitOps-CICD"
                    sh "git add ."
                    sh 'git commit -m "Update image version for Build - ${VERSION}"'
                    sh "git push origin master"
                }
            }
        }
    }
}