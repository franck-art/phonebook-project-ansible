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
            agent none
            stages {
               stage("Install ansible role dependencies") {
            agent any
                   steps {
                       sh 'ansible-galaxy install  -r roles/requirements.yml'
                   }
               }
                stage("Ping targeted hosts") {
             agent any
                   steps {
                       sh 'ansible all -m ping -i hosts --private-key id_rsa'
                   }
                }

                 stage("Vérify ansible playbook syntax") {
                 //agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
                   agent any
                   steps {
                       sh 'sudo yum install epel-release -y'
                       sh 'sudo yum install python-pip -y'
                       sh 'sudo pip install ansible-lint'
                       sh 'ansible-lint -x 306 playbook/phonebook.yml'
                       sh 'echo "${GIT_BRANCH}"'
                   }
                 } 
               stage("Build docker images on build host") {
              agent any
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                   }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "build" --limit build playbook/phonebook.yml'
                   }
               }


               stage("Scan docker images on build host") {
               agent any
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                  }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --limit build playbook/clair-scan.yml'
                   }

               }
                stage("Push on docker hub") {
              agent any
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                   }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "push" --limit build playbook/phonebook.yml'
                   }
               }
 

               stage("Deploy app in Pre-production Environment") {
               agent any
                    when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "preprod" --limit preprod playbook/phonebook.yml'
                   }
               }


               stage("Test the functioning of the app in Preprod environment") {
               agent any
                    when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "test" --limit preprod playbook/phonebook.yml'
                   }
               }

            }

         }
               stage("Installation of configuration files") {
               agent any
                    when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts  playbook/security_test.yml'
                   }
               }
 

               stage("Test the performance of the app with JMETER in Preprod environment") {

                agent any
                     when {
                       expression { GIT_BRANCH == 'origin/dev' }
                    }
                   steps {
                       echo '' > ${WORKSPACE}/docker-jmeter/docker-jmeter/report.jtl
                       sh '${WORKSPACE}/docker-jmeter/./run.sh -n -t ${WORKSPACE}/docker-jmeter/plan_test_jmeter.jmx  -l ${WORKSPACE}/docker-jmeter/docker-jmeter/report.jtl'
                       perfReport '${WORKSPACE}/docker-jmeter/docker-jmeter/report.jtl'
                       perfReport errorFailedThreshold: 50, errorUnstableThreshold: 50, filterRegex: '', sourceDataFiles: '${WORKSPACE}/docker-jmeter/docker-jmeter/report.jtl'
                   }
               }

               stage("Deploy app in Production Environment") {
                    
              // agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
                 agent any
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {

                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "prod" --limit prod playbook/phonebook.yml'
                  
                   }
               }

            
         

           stage('Find xss vulnerability') {
            agent { docker { 
                  image 'gauntlt/gauntlt' 
                  args '--entrypoint='
                  } }
            steps {
                sh 'gauntlt --version'
                sh 'gauntlt /tmp/xss.attack'
            }
          }
         

/*          stage('Find Nmap vulnerability') {
            agent { docker {
                  image 'gauntlt/gauntlt'
                  args '--entrypoint='
                  } }
            steps {
                sh 'gauntlt --version'
                sh 'gauntlt /tmp/nmap.attack'
            }
          }


          stage('Find Os detection vulnerability') {
            agent { docker {
                  image 'gauntlt/gauntlt'
                  args '--entrypoint='
                  } }
            steps {
                sh 'gauntlt --version'
                sh 'gauntlt /tmp/os_detection.attack'
            }
          }
*/
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
