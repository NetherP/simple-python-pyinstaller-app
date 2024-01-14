pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.12.1-alpine3.19'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage ('Deploy') {
            steps {
                script {
                    def dockerCmd = 'touch /tmp/testdeploy'
                    sshagent(['b000e456-633b-41b7-8953-17eb7343f3c8']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.92.144.96 ${dockerCmd}"
                    }
                }
            }
        }
        
    }
}