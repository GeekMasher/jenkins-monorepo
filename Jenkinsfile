pipeline {
    agent any
    stages {
        stage('Build and Analyse Stage') {
            failFast true
            parallel {
                stage('App') {
                    steps {
                        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                            sh "codeql-runner init --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth ${GITHUB_TOKEN} --queries security-and-quality"
                            sh ". /srv/checkout/myrepository/codeql-runner/codeql-env.sh"

                            // Build
                            sh "dotnet build -c Release ./WebApp/WebApp.csproj"

                            sh "codeql-runner analyze --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth ${GITHUB_TOKEN}  --commit $(git rev-parse HEAD) --ref refs/heads/$(git branch --show-current)"
                        }
                    }
                }
                stage('Api') {
                    steps {
                        sh "codeql-runner init --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth ${GITHUB_TOKEN} --queries security-and-quality"
                        sh ". /srv/checkout/myrepository/codeql-runner/codeql-env.sh"

                        // Build
                        sh "dotnet build -c Release ./WebAPI/WebAPI.csproj"
                        
                        sh "codeql-runner analyze --repository GeekMasher/jenkins-monorepo --github-url https://github.com --github-auth ${GITHUB_TOKEN}  --commit $(git rev-parse HEAD) --ref refs/heads/$(git branch --show-current)"
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
