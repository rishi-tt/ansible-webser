pipeline {
    agent {label "agentfarm"}
    stages {
        stage('Delete the workspace') {
	    steps {
	        cleanWs()
	    }
	}
	stage('Installing Ansible') {
	    steps {
	      script {
	        def ansible_exists = fileExists '/usr/bin/ansible'
		if (ansible_exists == true) {
		     echo "Skipping Ansible install - already exists"
		} else {
	       sh 'sudo apt-get update -y && sudo apt-get upgrade -y'
	       sh 'sudo apt install -y wget tree unzip ansible python3-pip python3-apt'
	        }
	       }
	      }
	}
	stage('Download Ansible Code') {
	    steps {
	        git credentialsId: 'git-repo-creds', url: 'git@github.com:gurcharan01/ansible-webserver.git'
	    }
	}
      }
}

