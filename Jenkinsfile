node {
    stage ('Build') {
        docker.image('python:3.12.1-alpine3.19').inside {
            sh 'pwd'
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py'
        }
    }
}