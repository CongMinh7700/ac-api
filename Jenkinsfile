pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME = "C:\\Program Files\\dotnet"
        PATH = "${DOTNET_CLI_HOME};${env.PATH}"
        DEPLOY_PATH = 'ac-api/publish'
    }

    stages {
        stage('Clone Source') {
            steps {
                git credentialsId: 'CongMinh7700', 
                    url: 'https://github.com/CongMinh7700/ac-api.git', 
                    branch: 'main'
            }
        }

        stage('Restore Dependencies') {
            steps {
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                bat 'dotnet test --no-restore'
            }
        }

        stage('Publish') {
            steps {
                bat 'dotnet publish -c Release -o publish'
            }
        }

        stage('Archive Build') {
            steps {
                bat 'tar -czf api-publish.tar.gz -C publish .'
            }
        }

        stage('Upload Service Files') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'UbtServiceDev',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'weather-one.service',
                                        remoteDirectory: "${env.DEPLOY_PATH}"
                                    ),
                                    sshTransfer(
                                        sourceFiles: 'weather-two.service',
                                        remoteDirectory: "${env.DEPLOY_PATH}"
                                    )
                                ],
                                verbose: true
                            )
                        ]
                    )
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'UbtServiceDev',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'api-publish.tar.gz',
                                        removePrefix: '',
                                        remoteDirectory: "${env.DEPLOY_PATH}",
                                        execCommand: """
                                            cd ${env.DEPLOY_PATH} &&
                                            tar -xzf api-publish.tar.gz &&
                                            rm api-publish.tar.gz &&
                                            
                                            sudo mv weather-one.service /etc/systemd/system/weather-one.service &&
                                            sudo mv weather-two.service /etc/systemd/system/weather-two.service &&
                                            
                                            sudo systemctl daemon-reload &&
                                            sudo systemctl enable weather-one.service &&
                                            sudo systemctl enable weather-two.service &&
                                            sudo systemctl restart weather-one.service &&
                                            sudo systemctl restart weather-two.service
                                        """.stripIndent(),
                                        execTimeout: 300000
                                    )
                                ],
                                verbose: true
                            )
                        ]
                    )
                }
            }
        }
    }

    post {
        success {
            echo '✅ API Deployment successful!'
        }
        failure {
            echo '❌ API Deployment failed!'
        }
        always {
            cleanWs()
        }
    }
}
