pipeline {
    agent any
    stages {
        stage ('Daily Compliance Run') {
            steps {
                echo 'Running automated hardening and compliance scan via SSH on localhost...'
                script {
                    def remote = [:]
                    remote.name = "localhost"
                    remote.host = "127.0.0.1" // Vi ansluter lokalt till Cloud Shell via SSH!
                    remote.allowAnyHosts = true
                    
                    withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                        remote.user = userName
                        remote.identityFile = identity
                        
                        stage ("Enforce with Ansible") {
                            // Vi kör installationen och playboken via SSH direkt på din host istället för i en container
                            sshCommand remote: remote, command: 'sudo apt-get update && sudo apt-get install ansible -y'
                            sshCommand remote: remote, command: 'mkdir -p ~/secops && git clone https://github.com/lftraining-lfs262/secops.git ~/secops || true'
                            sshCommand remote: remote, command: 'ansible-galaxy collection install devsec.hardening'
                            sshCommand remote: remote, command: 'cd ~/secops/ansible && ansible-playbook compliance.yaml'
                        }
                        
                        stage ("Scan with InSpec") {
                            // Vi kör inspec-kommandot på din host där det redan är installerat
                            sshCommand remote: remote, command: 'inspec exec --no-distinct-exit /root/linux-baseline/'
                        }
                    }
                }
            }
        }
    }
}
