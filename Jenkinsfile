pipeline {
    agent any
    
    tools {
        nodejs 'Nodejs16'  // Tên đã cấu hình ở trên
    }
    
    environment {
        BUILD_START_TIME = ''
    }
    
    stages {
        stage('Check Files') {
    steps {
        sh 'pwd'
        sh 'ls -la'
        dir('ecomResource') {
            sh 'ls -la'
        }
    }
}
        stage('Check Docker') {
            steps {
                sh 'whoami'   // Kiểm tra user đang chạy Jenkins
                sh 'docker -v'
                sh 'docker-compose -v'
            }
        }
         stage('Build Docker Images') {
    steps {
        dir('ecomResource') {  // Di chuyển vào thư mục chứa docker-compose.yml
            sh 'docker-compose build'
        }
    }
}

stage('Run Containers') {
    steps {
        dir('ecomResource') {
            sh 'docker-compose up -d'
        }
    }
}

        stage('Check Running Containers') {
            steps {
                script {
                    sh 'docker ps'
                }
            }
        }
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
                    
                   dir('ecomResource/eCommerce_Reactjs') {  
                        sh 'node -v'  // Kiểm tra version
                        sh 'npm -v'   // Kiểm tra npm
                        sh 'npm install'
                        sh 'npm test --passWithNoTests || true'
                        sh 'CI=false npm run build'

                    }
                    
                    def duration = System.currentTimeMillis() - startTime
                    sh "echo ${duration} > metrics/frontend-build.txt"
                     writeFile file: 'ecomResource/metrics/frontend-build.txt', text: duration.toString()
                }
            }
        }
         stage('Backend Build & Test') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    
                    dir('ecomResource/ecomAPI') {  
                        sh 'node -v'  // Kiểm tra version Node.js
                        sh 'npm -v'   // Kiểm tra npm
                        sh 'npm install'
                        sh 'npx sequelize-cli db:migrate'
                        sh 'npm test --passWithNoTests || true'
                        sh 'CI=false npm run'
                    }
                    
                    def duration = System.currentTimeMillis() - startTime
                    writeFile file: 'ecomResource/metrics/backend-build.txt', text: duration.toString()
                }
            }
        }
        stage('Generate Report') {
            steps {
                script {
                    def report = """
                        Pipeline Report
                        ==============
                         Backend Build: ${readFile('ecomResource/metrics/backend-build.txt').trim()}ms
                        Frontend Build: ${readFile('ecomResource/metrics/frontend-build.txt').trim()}ms
                    """.stripIndent()
                    
                    writeFile file: 'pipeline-report.md', text: report
                }
            }
        }
        stage('Monitor Resources') {
    steps {
        script {
            // CPU Usage
            def cpuUsage = sh(script: 'top -bn1 | grep "Cpu(s)" | sed "s/.*, *\\([0-9.]*\\)%* id.*/\\1/" | awk \'{print 100 - $1}\'', returnStdout: true).trim()
            
            // Memory Usage
            def memUsage = sh(script: 'free -m | awk \'NR==2{printf "%.2f", $3*100/$2 }\'', returnStdout: true).trim()
            
            // Disk Usage
            def diskUsage = sh(script: 'df -h / | awk \'NR==2{print $5}\'', returnStdout: true).trim()
            
            // Lưu metrics
            writeFile file: 'ecomResource/metrics/resources.txt', text: """
                CPU: ${cpuUsage}%
                Memory: ${memUsage}%
                Disk: ${diskUsage}
            """
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
