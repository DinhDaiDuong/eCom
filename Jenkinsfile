pipeline {
    agent any
    
    tools {
        nodejs 'Nodejs16' 
    }
    
    environment {
        BUILD_START_TIME = ''
       TEST_SIZES = '1 10 50'
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
                    
                      dir('ecomResource/eCommerce_Reactjs') {  
                        sh 'node -v'  // Kiểm tra version
                        sh 'npm -v'   // Kiểm tra npm
                         sh 'npm install --legacy-peer-deps'  
                        sh 'npm test --passWithNoTests || true'
                        sh 'CI=false npm run build'

                    }
                    
                    def duration = System.currentTimeMillis() - startTime
                    sh "echo ${duration} > metrics/frontend-build.txt"
                     writeFile file: 'ecomResource/metrics/frontend-build.txt', text: duration.toString()
                }
            }
        }
        
        stage('Generate Report') {
            steps {
                script {
                    def report = """
                        Pipeline Report
                        ==============
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
            writeFile file: 'eCommerce/metrics/resources.txt', text: """
                CPU: ${cpuUsage}%
                Memory: ${memUsage}%
                Disk: ${diskUsage}
            """
        }
    }
}
  stage('Run Tests') {
            steps {
                script {
                    // Function to run test with specific job count
                    def runTest = { jobCount ->
                        def startTime = System.currentTimeMillis()
                        def initialCpu = sh(script: "top -bn1 | grep 'Cpu(s)' | awk '{print \$2}'", returnStdout: true).trim()
                        def initialRam = sh(script: "free -m | awk '/Mem:/ {print \$3}'", returnStdout: true).trim()
                        
                        // Run jobs in parallel
                        def parallel_jobs = [:]
                        for (int i = 0; i < jobCount.toInteger(); i++) {
                            def job_num = i
                            parallel_jobs["job_${job_num}"] = {
                                sh 'sleep 10'
                            }
                        }
                        parallel parallel_jobs
                        
                        def endTime = System.currentTimeMillis()
                        def finalCpu = sh(script: "top -bn1 | grep 'Cpu(s)' | awk '{print \$2}'", returnStdout: true).trim()
                        def finalRam = sh(script: "free -m | awk '/Mem:/ {print \$3}'", returnStdout: true).trim()
                        
                        // Calculate metrics
                        def duration = (endTime - startTime) / 1000
                        def cpuUsage = finalCpu.toFloat() - initialCpu.toFloat()
                        def ramUsage = finalRam.toInteger() - initialRam.toInteger()
                        
                        // Save metrics
                        sh """
                            echo "${duration},${cpuUsage},${ramUsage}" > metrics/job_${jobCount}.txt
                        """
                        
                        echo "Results for ${jobCount} jobs:"
                        echo "Duration: ${duration}s"
                        echo "CPU Usage: ${cpuUsage}%"
                        echo "RAM Usage: ${ramUsage}MB"
                        
                        return [duration: duration, cpu: cpuUsage, ram: ramUsage]
                    }
                    
                    // Run tests for different job counts
                    def results = [:]
                    TEST_SIZES.split().each { size ->
                        results[size] = runTest(size)
                    }
                    
                    // Generate report
                    def report = """# Performance Results
                    
| Jobs | Duration (s) | CPU Usage (%) | RAM Usage (MB) |
|------|-------------|---------------|----------------|
"""
                    results.each { size, metrics ->
                        report += "| ${size} | ${metrics.duration} | ${metrics.cpu} | ${metrics.ram} |\n"
                    }
                    
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
