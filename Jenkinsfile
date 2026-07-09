pipeline {
    agent any
    stages {
        stage ('Daily Compliance Run') {
            steps {
                echo 'Running automated hardening and compliance scan on localhost...'
                script {
                    def remote = [:]
                    remote.name = "localhost"
                    remote.host = "127.0.0.1" // Vi kör lokalt mot Cloud Shell/Jenkins-hosten!
                    remote.allowAnyHosts = true
                    
                    withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                        remote.user = userName
                        remote.identityFile = identity
                        
                        stage ("Enforce with Ansible") {
                            // Vi ser till att paketen installeras och körs lokalt i din secops-mapp
                            sh 'ansible-galaxy collection install devsec.hardening'
                            sh 'cd ~/secops/ansible && ansible-playbook compliance.yaml'
                        }
                        
                        stage ("Scan with InSpec") {
                            // Vi kör inspec-kommandot direkt via Jenkins-agenten mot din lokala baslinje
                            sh 'inspec exec --no-distinct-exit /root/linux-baseline/'
                        }
                    }
                }
            }
        }
    }
}
