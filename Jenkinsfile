pipeline {
    agent any
    stages {
        stage('Build and Analyse Stage') {
            failFast true
            parallel {
                stage('App') {
                    steps {
                        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                            sh "codeql-runner init --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN --queries security-and-quality --temp-dir '.codeql-runner-app'"
                            sh ". ./.codeql-runner-app/codeql-env.sh"

                            // Build
                            sh "dotnet build -c Release ./WebApp/WebApp.csproj"

                            sh "codeql-runner analyze --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN  --commit $GIT_COMMIT --ref $GIT_BRANCH --temp-dir '.codeql-runner-app'"
                        }
                    }
                }
                stage('Api') {
                    steps {
                        
                        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                            sh "codeql-runner init --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN --queries security-and-quality --temp-dir '.codeql-runner-api'"
                            sh ". ./.codeql-runner-api/codeql-env.sh"

                            // Build
                            sh "dotnet build -c Release ./WebAPI/WebAPI.csproj"

                            sh "codeql-runner analyze --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN --commit $GIT_COMMIT --ref $GIT_BRANCH --temp-dir '.codeql-runner-api'"
                        }
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
