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
        stage('Deploy') { 
            agent any
            environment { 
                VOLUME = '/myapp:/src' 
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) { 
                    unstash(name: 'compiled-results') 
                    script {
                        def scpCmd = "scp -o StrictHostKeyChecking=no -r sources/* ec2-user@54.179.43.54:/myapp"

                        def dockerCmd = "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                        sshagent(['b000e456-633b-41b7-8953-17eb7343f3c8']) {

                            sh scpCmd
                            sh "ssh -o StrictHostKeyChecking=no ec2-user@54.179.43.54 \"${dockerCmd}\""
                        }
                    }
                }
            }
            post {
                success {
                    script {
                        def scpCmd = "scp -o StrictHostKeyChecking=no -r ec2-user@54.179.43.54:/myapp/dist/add2vals ${env.BUILD_ID}/sources/dist/add2vals"

                        def dockerCmd = "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                        sshagent(['b000e456-633b-41b7-8953-17eb7343f3c8']) {
                            sh scpCmd
                            sh "ssh -o StrictHostKeyChecking=no ec2-user@54.179.43.54 \"${dockerCmd}\""
                        }
                    }
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}