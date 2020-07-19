@Library('jenkins-shared-library') _


pipeline {
    agent none
    stages {
        stage('Check bash syntax') {
            agent { docker { image 'koalaman/shellcheck-alpine:latest' } }
            steps {
              script { bashCheck }
            }
        }
        stage('Check yaml syntax') {
            agent { docker { image 'sdesbure/yamllint' } }
            steps {
              script { yamlCheck }
            }
        }
         stage('Check markdown syntax') {
            agent { docker { image 'ruby:alpine' } }
            steps {
            script { markdownCheck }
            }
         }
         stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
                DEVOPSKEY = credentials('devopskey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
                sh 'cp \$DEVOPSKEY id_rsa'
                sh 'chmod 600 id_rsa'
            }
         }
         stage('Test and deploy the application') {
            agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
            stages {
               stage("Install ansible role dependencies") {
                   steps {
                       sh 'ansible-galaxy install -r requirements.yml'
                   }
               }
                stage("Ping targeted hosts") {
                   steps {
                       sh 'ansible all -m ping -i hosts --private-key id_rsa'
                   }
                }
                 stage("Vérify ansible playbook syntax") {
                   steps {
                       sh 'ansible-lint -x 306 playbook.yml'
                       sh 'echo "${GIT_BRANCH}"'
                   }
                 } 
               stage("Build docker images on build host") {
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                   }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "build" --limit build phonebook.yml'
                   }
               }
               stage("Deploy app in Pre-production Environment") {
                    when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "preprod" --limit preprod phonebook.yml'
                   }
               }


               stage("Test the functioning of the app in Preprod environment") {
                    when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "test" --limit preprod phonebook.yml'
                   }
               }

               stage("Deploy app in Production Environment") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "prod" --limit prod phonebook.yml'
                   }
               }

            }
         }
    }
    post {
     always {
       script {
         // Use slackNotifier.groovy from shared library and provide current build result as parameter 

         clean
         slackNotifier currentBuild.result
     }
    }

}   
}
