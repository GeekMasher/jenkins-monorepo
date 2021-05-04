pipeline {
    agent any

    environment {
        // GitHub
        GITHUB_DOMAIN     = "https://github.com"
        GITHUB_TOKEN      = credentials('github-token')
        GITHUB_REPOSITORY = "GeekMasher/jenkins-monorepo"
        GITHUB_REF        = "refs/heads/master"

        // CodeQL
        CODEQL_DATABASES = '.codeql'
        CODEQL_RESULTS   = '.codeql-results'
        CODEQL_SUITE     = 'code-scanning'
    }

    stages {
        stage('Init') {
            steps {
                // Just to make sure everything is clean for the `Build and Analyse Stage`
                sh "rm -r ${CODEQL_DATABASES} 2> /dev/null || true"
                sh "rm -r ${CODEQL_RESULTS} 2> /dev/null || true"

                sh "mkdir -p ${CODEQL_DATABASES} ${CODEQL_RESULTS}"
            }
        }

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
                        // CodeQL Init
                        sh "codeql database create \
                            --language=${CODEQL_LANGUAGE} \
                            --command=\"${BUILD_COMMAND}\" \
                            --search-path=\"$CODEQL_SEARCH_PATH\" \
                            ${CODEQL_DATABASE}"
                        sh "codeql database upgrade ${CODEQL_DATABASE}"

                        // Build and Analyze
                        sh "codeql database analyze \
                            --format=\"sarif-latest\" \
                            --search-path=\"$CODEQL_SEARCH_PATH\" \
                            --sarif-category="${STAGE_NAME}" \
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\" \
                            ${CODEQL_DATABASE} ${CODEQL_LANGUAGE}-${CODEQL_SUITE}"

                        sh "codeql github upload-results \
                            --repository="${GITHUB_REPOSITORY}" \
                            --commit="\$(git rev-parse HEAD)" \
                            --ref="${GITHUB_REF}" \
                            --sarif=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\""
                    }
                }
                stage('Api') {
                    environment {
                        // Build command
                        BUILD_COMMAND = "dotnet build -c Release ./WebAPI/WebAPI.csproj"
                        // CodeQL
                        CODEQL_DATABASE = "${CODEQL_DATABASES}/${STAGE_NAME}"
                        CODEQL_LANGUAGE = "csharp"
                    }
                    steps {
                        // CodeQL Init
                        sh "codeql database create \
                            --language=${CODEQL_LANGUAGE} \
                            --command=\"${BUILD_COMMAND}\" \
                            --search-path=\"$CODEQL_SEARCH_PATH\" \
                            ${CODEQL_DATABASE}"
                        sh "codeql database upgrade ${CODEQL_DATABASE}"

                        // Build and Analyze
                        sh "codeql database analyze \
                            --format=\"sarif-latest\" \
                            --search-path=\"$CODEQL_SEARCH_PATH\" \
                            --sarif-category="${STAGE_NAME}" \
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\" \
                            ${CODEQL_DATABASE} ${CODEQL_LANGUAGE}-${CODEQL_SUITE}"

                        sh "codeql github upload-results \
                            --repository="${GITHUB_REPOSITORY}" \
                            --commit="\$(git rev-parse HEAD)" \
                            --ref="${GITHUB_REF}" \
                            --sarif=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\""
                    }
                }
            }
        }
    }

    post {
        failure {
            sh "echo '${BUILD_ID}' > build-id.txt"
        }
    }
}
