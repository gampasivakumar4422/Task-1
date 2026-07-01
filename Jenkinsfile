pipeline {

    agent {
        kubernetes {
            label 'k8s-agent'
        }
    }

    tools {
        maven 'Maven3'
    }

    environment {
        SERVER_IP = '54.164.137.217'
        SSH_USER  = 'ec2-user'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gampasivakumar4422/Task-1'
            }
        }

        stage('Build WAR') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Verify WAR') {
            steps {
                container('maven') {
                    sh 'ls -lh target'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(credentials: ['tng-server-key']) {

                    sh '''
                    echo "========== Copy WAR =========="

                    scp -o StrictHostKeyChecking=no target/*.war ${SSH_USER}@${SERVER_IP}:/tmp/

                    echo "========== Deploy =========="

                    ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SERVER_IP} << EOF

                    sudo rm -rf /opt/tomcat/webapps/SivaKumar*
                    sudo rm -f /opt/tomcat/webapps/*.war

                    sudo cp /tmp/*.war /opt/tomcat/webapps/

                    echo "Deployment Successful"

                    EOF
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['tng-server-key']) {

                    sh '''
                    ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SERVER_IP} "ls -l /opt/tomcat/webapps"
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "=================================="
            echo "Application Deployed Successfully"
            echo "=================================="
        }

        failure {
            echo "=================================="
            echo "Deployment Failed"
            echo "=================================="
        }

        always {
            cleanWs()
        }
    }
}
