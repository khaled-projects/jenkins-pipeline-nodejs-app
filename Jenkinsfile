pipeline {
    agent any

    tools {
        nodejs 'NodeJS 23.9.0'  // The name configured in Global Tool Configuration
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
                            // Run OWASP Dependency Check
                            dependencyCheck additionalArguments: '''
                                --scan ./ 
                                --out ./${REPORT_DIR} 
                                --format ALL 
                                --prettyPrint
                            ''', odcInstallation: 'OWASP-Dependency-Check'
                            
                            
            
                        }
                    }
                stage('NPM Audit (Critical)') {
                    steps {
                        script {
                            sh 'npm audit --audit-level=critical'
                        }
                    }
                }
            }
        }
    }

    
}
