pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("mehrfatima/personal-portfolio:${env.BUILD_ID}")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html' // Simple check for index.html
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the new version
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "my_webserver", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull mehrfatima/personal-portfolio:${env.BUILD_ID}
                                        docker stop personal-portfolio-container || true
                                        docker rm personal-portfolio-container || true
                                        docker run -d --name personal-portfolio-container -p 80:80 mehrfatima/personal-portfolio:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'sp20-bcs-049@cuiatk.edu.pk',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}
