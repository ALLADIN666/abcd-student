pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/ALLADIN666/abcd-student', branch: 'main'
                }
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        stage('DAST') {
            steps{
                sh 'docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop;sleep 5'
                sh '''
                    docker run --name zap -v /root/abc/abc-student/.zap:/zap/wrk/:rw -t ghrc.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd addoninstall communityScripts -addoninstall pscanrules; zap-full-scan.py -t http://localhost:3000
                '''
                '''
    }
}
