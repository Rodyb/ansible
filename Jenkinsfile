pipeline {
    agent any
    environment {
        DOCKER_PASSWORD = '' // Initialize as empty, will be set dynamically
        ANSIBLE_SERVER = "161.35.88.101"
    }
    stages {
        stage("Copy files to Ansible server") {
            steps {
                script {
                    echo "Copying all necessary files to Ansible control node"
                    sshagent(['ansible-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@${ANSIBLE_SERVER}:/root"
                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            sh "scp $keyfile root@$ANSIBLE_SERVER:/root/ssh-key.pem && ssh -o StrictHostKeyChecking=no root@${ANSIBLE_SERVER} 'chmod 600 /root/ssh-key.pem'"
                        }

                    }
                }
            }
        }
        stage("Execute Ansible playbook") {
            steps {
                script {
                    echo "Calling Ansible playbook to configure EC2 instances"

                    withCredentials([string(credentialsId: 'docker-password-secret', variable: 'DOCKER_PASSWORD')]) {
                        def remote = [:]
                        remote.name = "ansible-server"
                        remote.host = ANSIBLE_SERVER
                        remote.allowAnyHosts = true

                        withCredentials([sshUserPrivateKey(credentialsId: 'ansible-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            remote.user = user
                            remote.identityFile = keyfile
//                             sshScript remote: remote, script: "prepare_ansible_server.sh"
//                             sshCommand remote: remote, command: "ansible-playbook my-playbook.yaml"
                                sshCommand remote: remote, command: "ansible-playbook my-playbook.yaml -e \"docker_password=${DOCKER_PASSWORD}\""

                        }
                    }
                }
            }
        }
    }
}
