def retryCount = 0
def maxRetries = 2

pipeline {
    agent any
    
    options {
        timestamps()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Test') {
            steps {
                script {
                    try {
                        // Attempt build and test
                        sh 'mvn clean install'
                    } catch (Exception e) {
                        // Log the failure
                        echo "Build failed on attempt ${retryCount + 1}"
                        echo "Error: ${e.getMessage()}"
                        
                        // Implement retry logic
                        while (retryCount < maxRetries) {
                            retryCount++
                            echo "Waiting 30 seconds before retry ${retryCount}..."
                            sleep 30
                            
                            try {
                                echo "Retry attempt ${retryCount}"
                                sh 'mvn clean install'
                                // If successful, break the retry loop
                                break
                            } catch (Exception retryError) {
                                echo "Retry ${retryCount} failed: ${retryError.getMessage()}"
                                if (retryCount >= maxRetries) {
                                    throw retryError
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    
    post {
        failure {
            script {
                def consoleLog = currentBuild.rawBuild.getLog(1000).join('\n')
                
                emailext (
                    subject: "Build Failed: ${currentBuild.fullDisplayName}",
                    body: """Build failed after ${retryCount + 1} attempts.
                    
                    Project: ${env.JOB_NAME}
                    Build Number: ${env.BUILD_NUMBER}
                    Build URL: ${env.BUILD_URL}
                    
                    Console Log (last 1000 lines):
                    ${consoleLog}
                    """,
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                    to: 'team@example.com'
                )
            }
        }
        success {
            echo "Build successful${retryCount > 0 ? ' after ' + retryCount + ' retries' : ''}!"
        }
    }
}
