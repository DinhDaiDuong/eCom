pipeline {
    agent any
    
    environment {
        BUILD_START_TIME = ''
    }
    
    stages {
        stage('Setup Metrics Directory') {
            steps {
                sh 'mkdir -p metrics'
            }
        }
        
        stage('Start Tracking') {
            steps {
                script {
                    BUILD_START_TIME = System.currentTimeMillis()
                }
            }
        }
        
        stage('Frontend Build') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    
                    dir('frontend') {
                        sh 'npm install'
                        sh 'npm test'
                        sh 'npm run build'
                    }
                    
                    def duration = System.currentTimeMillis() - startTime
                    sh "echo ${duration} > metrics/frontend-build.txt"
                }
            }
        }
        
        // Các stage khác tương tự
        
        stage('Generate Report') {
            steps {
                script {
                    def report = """
                        Pipeline Report
                        ==============
                        Frontend Build: ${readFile('metrics/frontend-build.txt').trim()}ms
                        Resources: ${readFile('metrics/resources.txt')}
                    """.stripIndent()
                    
                    writeFile file: 'pipeline-report.md', text: report
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'metrics/**, pipeline-report.md', fingerprint: true
        }
    }
}