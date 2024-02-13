pipeline {
    agent none
    stages {
        stage('Check yaml syntax') {
            agent { docker { image 'sdesbure/yamllint' } }
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
                EC2_SSH_KEY = credentials('ec2_ssh_key') 
                 EC2_HOST = '3.231.162.90' // Remplacez par l'adresse IP publique ou le DNS de votre instance EC2
                 EC2_USER = 'ubuntu' // Remplacez par l'utilisateur SSH de votre instance EC2
            }
            agent { docker { image 'registry.gitlab.com/carlinfongang-labs/docker-images/docker-ansible:latest' } }
            stages {
               //stage("Verify ansible playbook syntax") {
                   //steps {
                       //sh 'ansible-lint deploy.yml'
                   //}
              // }
               stage("Deploy app in production") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh '''
                       apt update
                       apt-get install -y sshpass
                       apt install ansible -y                
                       ansible-playbook -i hosts.yml --vault-password-file vault.key  --extra-vars "ansible_sudo_pass=$SUDOPASS" deploy.yml --private-key $EC2_SSH_KEY
                       '''
                   }
               } 
            }
          }
      }
    }
