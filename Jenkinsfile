pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    stages {

        stage('Nettoyage Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Clonage') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/AnassHadioui/CI_Project.git'
            }
        }

        stage('Verification Python') {
            steps {
                bat 'python --version'
                bat 'python -m pip --version'
            }
        }

        stage('Installation des dependances') {
            steps {
                bat 'python -m pip install -r requirements.txt'
            }
        }

        stage('Analyse qualite code - Pylint') {
            steps {
                bat 'python -m pylint calculator.py'
            }
        }

        stage('Tests unitaires') {
            steps {
                bat 'python -m pytest -v --ignore=Release'
            }
        }

        stage("Creation de l'artefact") {
            steps {
                bat '''
                    if exist Release rmdir /S /Q Release
                    if exist Release.zip del /Q Release.zip

                    mkdir Release

                    copy calculator.py Release
                    copy test_calculator.py Release
                    copy requirements.txt Release

                    powershell -Command "Compress-Archive -Path Release/* -DestinationPath Release.zip -Force"
                '''

                archiveArtifacts artifacts: 'Release.zip',
                                 fingerprint: true
            }
        }

        stage("Publication de l'artefact") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    bat '''
                        @echo off
                        curl.exe --fail --show-error ^
                            -u "%NEXUS_USER%:%NEXUS_PASSWORD%" ^
                            --upload-file Release.zip ^
                            http://localhost:8081/repository/python-release/Release.zip
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline termine'
        }

        success {
            echo 'BUILD SUCCESS'
        }

        failure {
            echo 'BUILD FAILED'
        }
    }
}