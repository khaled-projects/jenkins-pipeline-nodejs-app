pipeline {
    agent any

    tools {
        nodejs 'NodeJS 23.9.0'
    }

    environment {
        /* Referencing credentials from Jenkins */
        MONGO_URI = "mongodb://10.0.2.15:27017"
        BITBUCKET_COMMON_CREDS = credentials('mongo-credentials')
    }

    stages {
        stage('Verify Node.js and NPM') {
            steps {
                script {
                    sh 'node -v'
                    sh 'npm -v'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'npm install --no-audit'
                }
            }
        }

        stage('Security Checks') {
            parallel {
                stage('Dependency Check (OWASP 10)') {
                    steps {
                        script {
                            dependencyCheck additionalArguments: "--scan ./src --out ./ --format HTML --disableAssembly --disableJar", 
                                odcInstallation: 'OWASP-Dependency-Check'
                            dependencyCheckPublisher failedTotalCritical: 1, stopBuild: true
                        }
                    }
                }

                stage('NPM Audit (Critical)') {
                    options {
                        timestamps()
                    }
                    steps {
                        script {
                            sh 'npm audit --audit-level=critical'
                        }
                    }
                }
            }
        }

        stage('JUnit Tests') {
            steps {
                timeout(time: 1, unit: 'MINUTES') { // Set timeout to 1 minute
                        script {
                            echo "Connecting to MongoDB with user: $BITBUCKET_COMMON_CREDS"
                            echo " user - $BITBUCKET_COMMON_CREDS_USR "
                            echo " password - $BITBUCKET_COMMON_CREDS_PSW "
                            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                                sh 'npm test' // Runs the test script from package.json
                                junit allowEmptyResults: true, testResults: 'test-result.xml'
                            }
                        }
                }
            }
        }

        stage('Publish HTML Report') {
         steps {
             script {
                 echo "Checking coverage report files..."
                 sh 'ls -R coverage' // Debugging: Check if the report exists
                 sh 'npm run coverage'
                 catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS' , message: 'error will be fixed later') {
                     echo "Publishing HTML Report..."
                 publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, icon: '', keepAll: false, 
                              reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', 
                              reportName: 'Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
    }
}
    }
}
