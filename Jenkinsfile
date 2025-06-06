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

        // 🔁 Tách upload file service riêng ra trước
        stage('Upload Service File') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'UbtServiceDev',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'ac-api.service',
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

        // 🔁 Sau đó mới upload build + deploy
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
                                            sudo mv ac-api.service /etc/systemd/system/ac-api.service &&
                                            sudo systemctl daemon-reload &&
                                            sudo systemctl enable ac-api.service &&
                                            sudo systemctl restart ac-api.service
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
            echo 'API Deployment successful!'
        }
        failure {
            echo 'API Deployment failed!'
        }
        always {
            cleanWs()
        }
    }
}
