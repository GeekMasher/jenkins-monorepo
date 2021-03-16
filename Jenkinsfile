pipeline {
    agent any
    stages {
        stage('Build and Analyse Stage') {
            failFast true
            parallel {
                stage('App') {
                    steps {
                        sh "dotnet build -c Release ./WebApp/WebApp.csproj"
                    }
                }
                stage('Api') {
                    steps {
                        sh "dotnet build -c Release ./WebAPI/WebAPI.csproj"
                    }
                }
            }
        }
        stage('Upload') {
            steps {
                echo "Placeholder..."
            }
        }
    }
}