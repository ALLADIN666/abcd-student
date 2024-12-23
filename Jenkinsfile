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
                    git credentialsId: 'github-pat', url: 'https://github.com/ALLADIN666/abcd-student/', branch: 'main'
                }
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        stage('OSV') {
            steps {
                sh 'osv-scanner --format json --output results/osv_json_report.json -L package-lock.json || true'
            }
        }
         stage('Secrets') {
            steps {
                sh 'trufflehog git file://. --branch main --force-skip-archives --json > results/trufflehog_json_report.json'
            }
        }
        stage('SAST') {
            steps {
                sh ' semgrep scan --config auto --json -q  --no-autofix . > results/semgrep_json_report.json'
            }
        }
        stage('DAST') {
            steps{
                  sh 'docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop;sleep 5'
                  sh '''
                      docker run --name zap \
                      -v /root/abc/abcd-student/.zap:/zap/wrk/:rw \
                      -t ghcr.io/zaproxy/zaproxy:stable \
                      bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                  '''
            }
            post {
                always {
                    // ${WORKSPACE} resolves to /var/jenkins_home/workspace/ABCD
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop juice-shop
                        docker stop zap
                        docker rm zap
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending to DefectDojo...'
            withCredentials([string(credentialsId: 'email2dojo', variable: 'email2dojo')]) {
              defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: email2dojo)
              defectDojoPublisher(artifact: 'results/osv_json_report.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: email2dojo)
              defectDojoPublisher(artifact: 'results/trufflehog_json_report.json', productName: 'Juice Shop', scanType: 'Trufflehog Scan', engagementName: email2dojo)
              defectDojoPublisher(artifact: 'results/semgrep_json_report.json', productName: 'Juice Shop', scanType: 'Semgrep JSON Report', engagementName: email2dojo)
            }
        }
    }
}
            
