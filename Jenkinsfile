def runAndLog(String command) {
    sh """
        #!/usr/bin/env bash
        set +e
        touch log.txt
        (
            while true; do
                echo "[heartbeat] \$(date)"
                sleep 30
            done
        ) &
        HEARTBEAT_PID=\$!

        ${command} 2>&1 | tee -a log.txt
        STATUS=\${PIPESTATUS[0]}

        kill "\$HEARTBEAT_PID" 2>/dev/null || true
        wait "\$HEARTBEAT_PID" 2>/dev/null || true
        exit "\$STATUS"
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
