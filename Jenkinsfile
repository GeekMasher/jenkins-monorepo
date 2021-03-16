pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                sh 'GIT_BRANCH="refs/heads/\$(git branch --show-current)"'
                echo "Git Branch :: $GIT_BRANCH"
                sh "GIT_HASH=\$(git rev-parse HEAD)"
                echo "Git Hash :: $GIT_HASH"
            }
        }
        stage('Build and Analyse Stage') {
            failFast true
            parallel {
                stage('App') {
                    steps {
                        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                            sh "codeql-runner init --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN --queries security-and-quality"
                            sh ". ./codeql-runner/codeql-env.sh"

                            // Build
                            sh "dotnet build -c Release ./WebApp/WebApp.csproj"

                            sh "codeql-runner analyze --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN  --commit $GIT_HASH --ref $GIT_BRANCH"
                        }
                    }
                }
                stage('Api') {
                    steps {
                        
                        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                            sh "codeql-runner init --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN --queries security-and-quality"
                            sh ". ./codeql-runner/codeql-env.sh"

                            // Build
                            sh "dotnet build -c Release ./WebAPI/WebAPI.csproj"

                            sh "codeql-runner analyze --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth $GITHUB_TOKEN --commit $GIT_HASH --ref $GIT_BRANCH"
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
