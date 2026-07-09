pipeline {
    agent any
    stages {
        stage ('Daily Compliance Run') {
            steps {
                echo 'Running automated hardening and compliance scan via Containerized SSH on GKE Node...'
                script {
                    def remote = [:]
                    remote.name = "gke-node"
                    remote.host = "136.115.77.101" // Din fungerande GKE-nod!
                    remote.allowAnyHosts = true
                    
                    withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                        remote.user = userName
                        remote.identityFile = identity
                        
                        stage ("Enforce with Ansible") {
                            echo "Running Ansible inside a Docker container on the host..."
                            // Vi laddar ner ditt repo till nodens temporära katalog
                            sshCommand remote: remote, command: 'mkdir -p /tmp/secops && git clone https://github.com/lftraining-lfs262/secops.git /tmp/secops || (cd /tmp/secops && git pull)'
                            
                            // Vi kör Ansible inuti en officiell docker-container så vi slipper apt-get!
                            sshCommand remote: remote, command: 'docker run --rm -v /tmp/secops:/secops -v /etc:/etc -w /secops/ansible cytopia/ansible ansible-galaxy collection install devsec.hardening && docker run --rm -v /tmp/secops:/secops -v /etc:/etc -w /secops/ansible cytopia/ansible ansible-playbook compliance.yaml'
                        }
                        
                        stage ("Scan with InSpec") {
                            echo "Running InSpec inside a Docker container on the host..."
                            // Vi kör InSpec via Chef:s officiella docker-container, så slipper vi ladda ner InSpec på värdmaskinen!
                            sshCommand remote: remote, command: 'docker run --rm -v /etc:/etc chef/inspec exec https://github.com/dev-sec/linux-baseline --no-distinct-exit'
                        }
                    }
                }
            }
        }
    }
}
