pipeline {
    agent any

    environment {
        // GitHub
        GITHUB_DOMAIN     = "https://github.com"
        GITHUB_TOKEN      = credentials('github-token')
        GITHUB_REPOSITORY = "GeekMasher/jenkins-monorepo"

        // CodeQL
        CODEQL_DATABASES = '.codeql'
        CODEQL_RESULTS   = '.codeql-results'
        CODEQL_SUITE     = 'code-scanning'
    }

    stages {
        stage('Init') {
            steps {
                // Just to make sure everything is clean for the `Build and Analyse Stage`
                sh "rm -r ${CODEQL_DATABASES}"
                sh "rm -r ${CODEQL_RESULTS}"

                sh "mkdir -p ${CODEQL_RESULTS}"
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
                        // Init
                        sh "mkdir -p ${CODEQL_DATABASE}"

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
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\" \
                            ${CODEQL_DATABASE} ${CODEQL_LANGUAGE}-${CODEQL_SUITE}"

                        // Stash for cross-agent support
                        stash includes: '${CODEQL_RESULTS}/**.sarif', name: 'sarif'
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
                        sh "mkdir -p ${CODEQL_DATABASE}"

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
                            --output=\"${CODEQL_RESULTS}/${STAGE_NAME}-${CODEQL_LANGUAGE}.sarif\" \
                            ${CODEQL_DATABASE} ${CODEQL_LANGUAGE}-${CODEQL_SUITE}"

                        // Stash for cross-agent support
                        stash includes: '${CODEQL_RESULTS}/**.sarif', name: 'sarif'
                    }
                }
            }
        }
        stage('Upload') {
            // Upload to GitHub
            steps {
                // Unstash SARIF files
                unstash 'sarif'
                // Currently using the Runner due to the CLI does not have the 
                // ability to upload folders of SARIF files.
                sh "codeql-runner upload \
                    --sarif-file ${CODEQL_RESULTS} \
                    --github-url ${GITHUB_DOMAIN} \
                    --repository ${GITHUB_REPOSITORY} \
                    --commit \$(git rev-parse HEAD) \
                    --ref refs/heads/testing"
                // TODO: Fix hardcoded branch ref
            }
        }
    }

    post {
        failure {
            sh "echo '${BUILD_ID}' > build-id.txt"
        }
    }
}
