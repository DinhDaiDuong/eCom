pipeline {
    agent any
    
    tools {
        nodejs 'Nodejs16'  // Tên đã cấu hình ở trên
    }
    
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
                    
                    dir('eCommerce_Reactjs') {
                        sh 'node -v'  // Kiểm tra version
                        sh 'npm -v'   // Kiểm tra npm
                        sh 'npm install'
                        sh 'npm test'
                        sh 'npm run build'
                    }
                    
                    def duration = System.currentTimeMillis() - startTime
                    sh "echo ${duration} > metrics/frontend-build.txt"
                }
            }
        }
        
        stage('Generate Report') {
            steps {
                script {
                    def report = """
                        Pipeline Report
                        ==============
                        Frontend Build: ${readFile('metrics/frontend-build.txt').trim()}ms
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