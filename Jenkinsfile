pipeline {
    agent {
        label 'ubuntu2204'
    }
    environment {
        GITLAB_REPO_URL = 'https://gitlab.esss.lu.se/ecdc/dm-ansible.git'
        GITLAB_TOKEN = '_58bVWB3aXGBENqrzHTK'
        GITLAB_USERNAME = 'any_username'  // Use a placeholder username, as specified
        PROJECT = 'nightly-builds'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '1'))  // Keep only the latest build
    }
    stages {
        stage('Checkout GitLab Repo') {
            steps {
                script {
                    sh """
                    git clone https://${env.GITLAB_USERNAME}:${env.GITLAB_TOKEN}@${env.GITLAB_REPO_URL}
                    """
                }
            }
        }

        stage('Check Python version::debug') {
            steps {
                script {
                    dir("${env.PROJECT}") {
                        sh """
                        which python
                        python --version
                        """
                    }
                }
            }
        }

        stage('Ansible Deployment') {
            steps {
                script {
                    dir("${env.PROJECT}") {
                        sh """
                        which ansible
                        ansible --version
                        ansible-playbook -i inventories/site -l efu0234 efu.yml
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()  // Clean workspace after the build
        }
    }
}
