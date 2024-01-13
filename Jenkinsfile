pipeline {
    agent none
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
        stage('Deliver') {
            agent {
                label 'application-cicd' 
            }
            environment {
                VOLUME = '/mnt/myapp:/src' 
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    script {
                        // Install Docker on the EC2 instance if needed
                        sh 'sudo amazon-linux-extras install docker'
                        sh 'sudo service docker start'

                        // Run the Docker command on the EC2 instance
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                    }
                }
            }
            post {
                success {
                    // Archive artifacts from the EC2 instance
                    archiveArtifacts "/mnt/myapp/sources/dist/add2vals"
                    // Clean up on the EC2 instance
                    script {
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                    }
                }
            }
        }
    }
}