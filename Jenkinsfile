pipeline {
    agent any
        
        tools {
            jdk "jdk 11"
            gradle "gradle 5.6"
        }
    stages {
        stage('Clone sources'){
            steps{
                git branch: 'AltoroJ-3.2', credentialsId: 'Gitlab_examplico', url: 'http://gitlab.examplico.com/root/altoroj.git'
            }
        }
        stage('Build'){
            steps{
                sh 'gradle build'
            }
        }
        stage('Pull ZeroNorth Integration Container'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'zn_phayes_demo', variable: 'zero_north_api')]) {
                        ZN_API="${env.zero_north_api}".toString()
                    }
                    
                    //Pull ZN integration container image
                    println '*-*-*\nPull ZN integration container image\n*-*-*'
                    def image = docker.image('zeronorth/integration:latest')
                    image.pull()
                    
                    println '*-*-*\nPull ZN SQ runner image\n*-*-*'
                    docker.withRegistry('', 'examplico_docker_hub'){
                        def sonar_runner_image = docker.image('zeronorth/sonarqube-agent-job-runner:latest')
                        sonar_runner_image.pull()
                    }
                    
                }
            }
        }
        stage('ZeroNorth Scan Orchestration OWASP Dependency'){
            steps{
                script{
                    println '*-*-*\nScan #1: owasp dependency v3\n*-*-*'
                    def image = docker.image('zeronorth/integration:latest')
                    image.inside("-e POLICY_ID=XMPRvLc0TaaQw9jpmVWZ8Q -v \"${WORKSPACE}/:/results\" -v \"$WORKSPACE:/code\" -e CYBRIC_API_KEY=${ZN_API}") {
                    sh "python /app/cybric.py"
                    }                     
                }
            }
        }
        stage('ZeroNorth Scan Orchestration WhiteSource'){
            steps{
                script{
                    println '*-*-*\nScan #2: WhiteSource\n*-*-*'
                    def image = docker.image('zeronorth/integration:latest')
                    image.inside("-e POLICY_ID=SU8uwnopQq6IrqmRnJq2Qg -v \"${WORKSPACE}/:/results\" -v \"$WORKSPACE:/code\" -e CYBRIC_API_KEY=${ZN_API}") {
                    sh "python /app/cybric.py"                        }                       
                }
            }
        }
        stage('ZeroNorth Scan Orchestration Sonarqube'){
            steps{
                script{
                    println '*-*-*\nScan #3: Sonarqube\n*-*-*' 
                    def image = docker.image('zeronorth/integration:latest')
                    image.inside("-e POLICY_ID=MCml30XGSr-LkU-u74_9Ig -e WORKSPACE=\"${WORKSPACE}\" -v \"${WORKSPACE}/:/results\" -v \"$WORKSPACE:/code\" -v /var/run/docker.sock:/var/run/docker.sock -e SONAR_JAVA_LIBRARY_DIR=\".\" -e SONAR_JAVA_BINARY_DIR=\".\" -e CYBRIC_API_KEY=${ZN_API}") {
                    sh "python /app/cybric.py"
                    }
                }
            }
        }
        stage('ZeroNorth Scan Orchestration Veracode'){
            steps{
                script{
                    println '*-*-*\nScan #4: Veracode\n*-*-*'
                    def image = docker.image('zeronorth/integration:latest')
                    image.inside("-e POLICY_ID=DKj3a6MuQaqQltJEHRST3g -v \"${WORKSPACE}/:/results\" -v \"$WORKSPACE:/code\" -e INCLUDE_FILES=/code/build/libs/*.war -e CYBRIC_API_KEY=${ZN_API}") {
                    sh "python /app/cybric.py"
                    }
                }
            }
        }
    }
}
