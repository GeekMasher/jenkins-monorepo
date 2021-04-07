pipeline {
    agent any

    environment {
        // GitHub
        GITHUB_TOKEN     = credentials('github-token')
        // CodeQL
        CODEQL_DATABASES = '.codeql'
        CODEQL_RESULTS   = '.codeql-results'
        CODEQL_SUITE     = 'code-scanning'
    }

    stages {
        stage('Build and Analyse Stage') {
            failFast true
            parallel {
                stage('App') {
                    steps {
                        // Init
                        sh "export CODEQL_DATABASE=${CODEQL_DATABASES}/${STAGE_NAME}"
                        sh "mkdir -p ${CODEQL_DATABASE} ${CODEQL_RESULTS}"

                        // CodeQL Init
                        sh "codeql database create --language=csharp ${CODEQL_DATABASE}"
                        sh "codeql database upgrade ${CODEQL_DATABASE}"

                        // Build and Analyze
                        sh "codeql database analßyze \
                            --format=\"sarif-latest\" \
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}.sarif\" \
                            --command=\"dotnet build -c Release ./WebApp/WebApp.csproj\" \
                            ${CODEQL_DATABASE} ${CODEQL_SUITE}"
                    }
                }
                stage('Api') {
                    steps {
                        // Init
                        sh "export CODEQL_DATABASE=${CODEQL_DATABASES}/${STAGE_NAME}"
                        sh "mkdir -p ${CODEQL_DATABASE} ${CODEQL_RESULTS}"

                        // CodeQL Init
                        sh "codeql database create --language=csharp ${CODEQL_DATABASE}"
                        sh "codeql database upgrade ${CODEQL_DATABASE}"

                        // Build and Analyze
                        sh "codeql database analyze \
                            --format=\"sarif-latest\" \
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}.sarif\" \
                            --command=\"dotnet build -c Release ./WebApp/WebApp.csproj\" \
                            ${CODEQL_DATABASE} ${CODEQL_SUITE}"
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

    post {
        failure {
            sh "echo '${BUILD_ID}' > build-id.txt"
        }
    }
}
