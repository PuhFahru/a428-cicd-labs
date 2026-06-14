def runAndLog(String command) {
    sh """
        set +e
        ${command} > command.log 2>&1
        STATUS=\$?
        cat command.log
        cat command.log >> log.txt
        rm command.log
        exit \$STATUS
    """
}

node {
    try {
        docker.image('node:lts-buster-slim').inside('-p 3001:3000') {
            withEnv(['CI=true']) {
                sh 'rm -f log.txt'

                stage('Build') {
                    runAndLog('npm install')
                    runAndLog('npm run build')
                }

                stage('Test') {
                    runAndLog('npm test -- --watchAll=false')
                }
            }
        }
    } finally {
        archiveArtifacts artifacts: 'log.txt', allowEmptyArchive: true
    }
}
