pipeline {
    agent any
    stages {
        stage ('Daily Compliance Run') {
            steps {
                echo 'Running automated hardening and compliance scan via Containerized SSH on GKE Node...'
                script {
                    def remote = [:]
                    remote.name = "gke-node"
                    remote.host = "136.115.77.101"
                    remote.allowAnyHosts = true
                    
                    withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                        remote.user = userName
                        remote.identityFile = identity
                        
                        stage ("Enforce with Ansible") {
                            echo "Running Ansible inside a Docker container on the host..."
                            sshCommand remote: remote, command: 'mkdir -p /tmp/secops && git clone https://github.com/lftraining-lfs262/secops.git /tmp/secops || (cd /tmp/secops && git pull)'
                            
                            // Vi har lagt till --ignore-certs här nedanför!
                            sshCommand remote: remote, command: 'docker run --rm -v /tmp/secops:/secops -v /etc:/etc -w /secops/ansible cytopia/ansible ansible-galaxy collection install devsec.hardening --ignore-certs && docker run --rm -v /tmp/secops:/secops -v /etc:/etc -w /secops/ansible cytopia/ansible ansible-playbook compliance.yaml'
                        }
                        
                        stage ("Scan with InSpec") {
                            echo "Running InSpec inside a Docker container on the host..."
                            sshCommand remote: remote, command: 'docker run --rm -v /etc:/etc chef/inspec exec https://github.com/dev-sec/linux-baseline --no-distinct-exit'
                        }
                    }
                }
            }
        }
    }
}
