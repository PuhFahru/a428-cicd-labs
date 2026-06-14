def runAndLog(String command) {
    sh """
        bash -lc 'set +e; ${command} 2>&1 | tee command.log; STATUS=\${PIPESTATUS[0]}; cat command.log >> log.txt; rm -f command.log; exit \$STATUS'
    """
}

node {
    try {
        stage('Checkout') {
            checkout scm
        }

        withEnv(['CI=true', 'NODE_OPTIONS=--openssl-legacy-provider']) {
            sh 'rm -f log.txt'

            stage('Build') {
                runAndLog('npm install --no-audit --no-fund')
                runAndLog('npm run build')
            }

            stage('Test') {
                runAndLog('npm test -- --watchAll=false')
            }
        }
    } finally {
        archiveArtifacts artifacts: 'log.txt', allowEmptyArchive: true
    }
}
