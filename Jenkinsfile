pipeline{
    agent {label "agentfarm"}
    environment{
        KEY_FILE = '/home/ubuntu/.ssh/vm-instance-key.pem'
        USER = 'ubuntu'
    }
    stages{
        stage('delete the workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Installing Ansible'){
            steps{
                script{
                    def ansible_exists = fileExists '/usr/bin/ansible'
                    if (ansible_exists == true){
                        echo "package already exists"
                }else {
                   sh 'sudo apt install -y wget tree unzip ansible python' 
                }
              
            }
          }
        }
        stage('Download Ansible Code'){
            steps{
                git credentialsId: 'git-repo-creds', url: 'git@github.com:ParulArinTech/ansible-webserver.git'
            }
        }
        stage('Run Ansible-lint against playbooks'){
            steps{
                sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 apache-install.yml'
                sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 website-update.yml'
                sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 website-test.yml'
            }
            
        }
        stage('send slack notification'){
            steps{
                slackSend color: 'warning', message: "Parul: Please approve ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.JOB_URL} | Open>)"
                
            }
            
        }
        stage('request input'){
            steps{
                input 'please approve / deny this build'
            }
        }
        stage('install apache & update website'){
            steps{
                sh 'export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/apache-install.yml'
                sh 'export ANSIBLE_ROLES_PATH=/opt/jenkins/workspace/ansible-pipeline/roles && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-update.yml'
                //sh 'export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/apache-install.yml'
               //sh 'export ANSIBLE_ROLES_PATH=/opt/jenkins/workspace/ansible-pipeline/roles && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-update.yml'
            }
        }
        stage('test website'){
            steps{
                sh 'export ANSIBLE_ROLES_PATH=/opt/jenkins/workspace/ansible-pipeline/roles && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-test.yml'
            }
        }
    }
     post{
        success{
            slackSend color: 'warning', message: "Build ${env.JOB_NAME} ${env.BUILD_NUMBER} was Successful!"
        }
        failure{
            slackSend color: 'warning', message: "Build ${env.JOB_NAME} ${env.BUILD_NUMBER} Failed!"
        }
    }
}
