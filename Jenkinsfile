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
                            echo "Simulating or verifying baseline with Ansible container..."
                            sshCommand remote: remote, command: 'mkdir -p /tmp/secops && git clone https://github.com/lftraining-lfs262/secops.git /tmp/secops || (cd /tmp/secops && git pull)'
                            
                            cfg_cmd = "docker run --rm -v /tmp/secops:/secops -w /secops/ansible cytopia/ansible ansible localhost -m ping"
                            sshCommand remote: remote, command: cfg_cmd
                        }
                        
                        stage ("Scan with InSpec") {
                            echo "Running InSpec inside a Docker container on the host..."
                            // Vi har lagt till -e CHEF_LICENSE="accept" här nedanför!
                            sshCommand remote: remote, command: 'docker run --rm -e CHEF_LICENSE="accept" chef/inspec exec https://github.com/dev-sec/linux-baseline --no-distinct-exit'
                        }
                    }
                }
            }
        }
    }
}
