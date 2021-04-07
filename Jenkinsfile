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
                    environment {
                        // Build command
                        BUILD_COMMAND = "dotnet build -c Release ./WebApp/WebApp.csproj"
                        // CodeQL
                        CODEQL_DATABASE = "${CODEQL_DATABASES}/${STAGE_NAME}"
                        CODEQL_LANGUAGE = "csharp"
                    }
                    steps {
                        // Init
                        sh "mkdir -p ${CODEQL_DATABASE} ${CODEQL_RESULTS}"

                        // CodeQL Init
                        sh "codeql database create --language=${CODEQL_LANGUAGE} ${CODEQL_DATABASE}"
                        sh "codeql database upgrade ${CODEQL_DATABASE}"

                        // Build and Analyze
                        sh "codeql database analyze \
                            --format=\"sarif-latest\" \
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\" \
                            --command=\"${BUILD_COMMAND}\" \
                            ${CODEQL_DATABASE} ${CODEQL_SUITE}"
                    }
                }
                stage('Api') {
                    environment {
                        // Build command
                        BUILD_COMMAND = "dotnet build -c Release ./WebApi/WebApi.csproj"
                        // CodeQL
                        CODEQL_DATABASE = "${CODEQL_DATABASES}/${STAGE_NAME}"
                        CODEQL_LANGUAGE = "csharp"
                    }
                    steps {
                        // Init
                        sh "mkdir -p ${CODEQL_DATABASE} ${CODEQL_RESULTS}"

                        // CodeQL Init
                        sh "codeql database create --language=${CODEQL_LANGUAGE} ${CODEQL_DATABASE}"
                        sh "codeql database upgrade ${CODEQL_DATABASE}"

                        // Build and Analyze
                        sh "codeql database analyze \
                            --format=\"sarif-latest\" \
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\" \
                            --command=\"${BUILD_COMMAND}\" \
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
