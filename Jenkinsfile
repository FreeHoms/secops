pipeline {
    agent any
    stages {
        stage ('Daily Compliance Run') {
            steps {
                echo 'Running automated hardening and compliance scan...'
                script {
                    def remote = [:]
                    remote.name = "localhost"
                    remote.host = "127.0.0.1"
                    remote.allowAnyHosts = true
                    
                    withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                        remote.user = userName
                        remote.identityFile = identity
                        
                        stage ("Enforce with Ansible") {
                            // Vi talar om för Jenkins att använda containern 'ansible' som har verktyget installerat!
                            container('ansible') {
                                sh 'ansible-galaxy collection install devsec.hardening'
                                sh 'cd ~/secops/ansible && ansible-playbook compliance.yaml'
                            }
                        }
                        
                        stage ("Scan with InSpec") {
                            // Vi kör inspec via dess fulla sökväg på hosten
                            sh 'inspec exec --no-distinct-exit /root/linux-baseline/'
                        }
                    }
                }
            }
        }
    }
}
