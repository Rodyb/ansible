pipeline {
    agent any
    stages {
        stage("copy files to ansible server") {
            steps {
                script {
                    echo "copying all necessary files to ansible control node"
                    sshagent(['ansible-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@167.172.38.180:/root"
                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                        sh 'scp $keyfile root@167.172.38.180:/root/ssh-key.pem'
                        }
                    }
                }
             stage("Execute ansible playbook") {
                    steps {
                        script {
                            echo "Calling ansible playbook to configure ec2 instances"
                            def remote = [:]
                            remote.name = "ansible-server"
                            remote.host = "167.172.38.180"
                            remote.allowAnyHosts = true

                            withCredentials([sshUserPrivateKey(credentialsId: 'ansible-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                                remote.user = user
                                remote.identityFile = keyfile
                                sshCommand remote: remote, command: "ls -l"
                            }
                }
            }
        }
    }
}
