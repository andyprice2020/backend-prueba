pipeline {
    agent {
        docker {
            image 'maven:3.8.1-adoptopenjdk-11' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Code') {
            steps {
                git 'https://github.com/andyprice2020/backend-prueba.git'
                sh 'mvn test'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Test-security') {
            steps {
                echo 'Testing...'
                snykSecurity(
                    snykInstallation: 'snyk-masti',
                    snykTokenId: 'token-snyk-masti',
                )
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package' 
            }
        }
        stage('Scanning Image') {
            steps {
                sysdig engineCredentialsId: 'sysdig-id', name: 'sysdig_secure_images', inlineScanning: true
            }
        }
        stage('Delivery') {
            steps {
                sh "docker run -p 8093:8000 -d --name web adminturneddevops/go-webapp-sample"
            }
        }
        stage('OWASP ZAP docker container') {
            steps {
                script {
                    echo "Pulling up OWASP ZAP container --> Start"
                    sh 'docker pull owasp/zap2docker-stable'
                    echo "Pulling up last VMS container --> End"
                    echo "Starting container --> Start"
                    sh """
                        docker run -dt --name owasp \
                        owasp/zap2docker-stable \
                        /bin/bash
                    """
                    echo "Container Started"
                    echo "Creating workspace directory"
                    sh """
                        docker exec owasp \
                        mkdir /zap/wrk
                    """
                }
            }
        }
        stage('OWASP ZAP Scanning target') {
            steps {
                script {
                    echo "Starting baseline scanning"
                    sh """
                        docker exec owasp \
                        zap-baseline.py \
                        -t $target \
                        -r report.html \
                        -I
                    """
                    echo "Copying scanning report"
                    sh '''
                        docker cp owasp:/zap/wrk/report.html ${WORKSPACE}/report.html
                    '''
                }
            }
        }
    }
    post{
        always {
            echo "Removing containers"
            sh '''
                docker stop web
                docker rm web
                docker stop owasp
                docker rm owasp
            '''
        }
    }
}