pipeline {
    agent none
    stages {
        stage('Check yaml syntax') {
            agent { 
                docker { 
                    image 'sdesbure/yamllint' 
                    args '-v /var/run/docker.sock:/var/run/docker.sock' 
                    } 
                }
            steps {
                sh 'yamllint --version'
                sh 'yamllint \${WORKSPACE}'
            }
        }
        stage('Check markdown syntax') {
            agent { docker { image 'ruby:alpine' } }
            steps {
                sh 'apk --no-cache add git'
                sh 'gem install mdl'
                sh 'mdl --version'
                sh 'mdl --style all --warnings --git-recurse \${WORKSPACE}'
            }
        }
        stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
            }
        }
        stage('Test and deploy the application') {
            environment {
                SUDOPASS = credentials('sudopass')
            }
            agent { docker { image image 'registry.gitlab.com/carlinfongang-labs/docker-images/docker-ansible:latest' } }
            stages {
               stage ("Update ansible-lint")
                    steps {
                        sh 'pip install --upgrade ansible-lint'
               stage("Verify ansible playbook syntax") {
                   steps {
                       sh 'ansible-lint deploy.yml'
                   }
               }
               stage("Deploy app in production") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh '''
                       apt-get update
                       apt-get install -y sshpass
                       ansible-playbook  -i hosts.yml --vault-password-file vault.key  --extra-vars "ansible_sudo_pass=$SUDOPASS" deploy.yml
                       '''
                   }
               } 
            }
          }
      }
    }
