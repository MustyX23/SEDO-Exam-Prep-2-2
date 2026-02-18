pipeline {
    agent any

    triggers {
        githubPush {
            onPush(true)
            onPullRequest(false)
        }
    }

    branches {
        only {
            main
        }
    }

    options {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        DOTNET_SKIP_FIRST_TIME_EXPERIENCE = '1'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out code from main branch..."
                }
                checkout scm
            }
        }

        stage('Restore Dependencies') {
            steps {
                script {
                    echo "Restoring NuGet dependencies..."
                }
                bat '''
                    dotnet restore HouseRentingSystem.sln
                '''
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building the application..."
                }
                bat '''
                    dotnet build HouseRentingSystem.sln --configuration Release --no-restore
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    echo "Running unit tests..."
                }
                bat '''
                    dotnet test HouseRentingSystem.UnitTests/HouseRentingSystem.UnitTests.csproj ^
                        --configuration Release ^
                        --no-build ^
                        --logger:"trx;LogFileName=unit-tests-results.trx" ^
                        --collect:"XPlat Code Coverage"
                '''
            }
        }

        stage('Run Integration Tests') {
            steps {
                script {
                    echo "Running integration tests..."
                }
                bat '''
                    dotnet test HouseRentingSystem.Tests/HouseRentingSystem.Tests.csproj ^
                        --configuration Release ^
                        --no-build ^
                        --logger:"trx;LogFileName=integration-tests-results.trx" ^
                        --collect:"XPlat Code Coverage"
                '''
            }
        }

        stage('Publish Test Results') {
            steps {
                script {
                    echo "Publishing test results..."
                }
                junit testResults: '**/TestResults/*.trx', allowEmptyResults: true
                publishHTML([
                    reportDir: 'TestResults',
                    reportFiles: 'index.html',
                    reportName: 'Code Coverage Report',
                    allowMissing: true
                ])
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up..."
            }
            cleanWs()
        }
        success {
            script {
                echo "Build and tests completed successfully!"
            }
        }
        failure {
            script {
                echo "Build or tests failed. Please check the logs."
            }
        }
        unstable {
            script {
                echo "Build is unstable. Some tests may have failed."
            }
        }
    }
}
