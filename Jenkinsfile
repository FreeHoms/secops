pipeline {
    agent any
    stages {
        stage ('Daily Compliance Run') {
            steps {
                echo 'Running automated hardening and compliance scan via Containerized SSH on GKE Node...'
                script {
                    def remote = [:]
                    remote.name = "gke-node"
                    remote.host = "136.115.77.101" [cite: 243, 257]
                    remote.allowAnyHosts = true [cite: 190, 268]
                    
                    withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) { [cite: 192, 269]
                        remote.user = userName [cite: 193, 270]
                        remote.identityFile = identity [cite: 194, 271]
                        
                        stage ("Enforce with Ansible") {
                            echo "Running Ansible inside a Docker container on the host..."
                            sshCommand remote: remote, command: 'mkdir -p /tmp/secops && git clone https://github.com/lftraining-lfs262/secops.git /tmp/secops || (cd /tmp/secops && git pull)'
                            
                            // Vi använder trippel-citattecken (''') för att skicka strängen exakt som den är till skalet utan escaping-strul
                            sshCommand remote: remote, command: '''docker run --rm -v /tmp/secops:/secops -v /etc:/etc -w /secops/ansible cytopia/ansible ansible-galaxy collection install devsec.hardening -p /secops/collections --ignore-certs && docker run --rm -v /tmp/secops:/secops -v /etc:/etc -e ANSIBLE_COLLECTIONS_PATH=/secops/collections -w /secops/ansible cytopia/ansible ansible-playbook compliance.yaml -e "{os_vars: {}}"'''
                        }
                        
                        stage ("Scan with InSpec") {
                            echo "Running InSpec inside a Docker container on the host..." [cite: 293, 416]
                            sshCommand remote: remote, command: 'docker run --rm -v /etc:/etc chef/inspec exec https://github.com/dev-sec/linux-baseline --no-distinct-exit'
                        }
                    }
                }
            }
        }
    }
}
