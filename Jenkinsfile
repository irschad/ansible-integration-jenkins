pipeline {   
    agent any
    environment {
        ANSIBLE_SERVER = "98.80.100.76"
    }
    stages {
        stage("copy files to ansible server") {
            steps {
                echo "copying all necessary files to ansible control node"
                sshagent(['ansible-server-key']) {
                    sh "scp -o StrictHostKeyChecking=no ansible/* ec2-user@${ANSIBLE_SERVER}:/home/ec2-user"
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                        sh 'scp ${keyfile} ec2-user@${ANSIBLE_SERVER}:/home/ec2-user/id_ed22519_ec2'
                    }

                }
            }
        }
        stage("execute ansible playbook") {
            steps {
                script {
                    echo "calling ansible server to configure ec2 instances"
                    def remote = [:]
                    remote.name = "server-one"
                    remote.host = env.ANSIBLE_SERVER
                    remote.allowAnyHosts = true

                    withCredentials([sshUserPrivateKey(credentialsId: 'ansible-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                        remote.user = user
                        remote.identityFile = keyfile
                        sshCommand remote: remote, command: "ansible-playbook ec2-playbook.yaml"
                    }
                    
                }
            }
        }
    }
} 
