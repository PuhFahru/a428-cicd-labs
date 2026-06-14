def logMessage(String message) {
    sh """
        touch log.txt
        cat <<'LOG_MESSAGE' | tee -a log.txt

${message}
LOG_MESSAGE
    """
}

def runAndLog(String command) {
    def shellScript = '''
        set +e
        touch log.txt
        (
            while true; do
                date '+[heartbeat] %Y-%m-%d %H:%M:%S %Z'
                sleep 30
            done
        ) &
        HEARTBEAT_PID=$!

        printf '\\n$ %s\\n' '__COMMAND__' >> log.txt
        __COMMAND__ >> log.txt 2>&1
        STATUS=$?
        printf '\\nCommand exit status: %s\\n' "$STATUS" >> log.txt
        cat log.txt

        kill "$HEARTBEAT_PID" 2>/dev/null || true
        wait "$HEARTBEAT_PID" 2>/dev/null || true
        exit "$STATUS"
    '''.replace('__COMMAND__', command)

    sh shellScript
}

node {
    def pipelineResult = 'SUCCESS'

    try {
        sh 'rm -f log.txt'
        logMessage('Pipeline started')
        logMessage("Job: ${env.JOB_NAME}")
        logMessage("Build: #${env.BUILD_NUMBER}")

        stage('Checkout') {
            logMessage('Stage started: Checkout')
            checkout scm
            runAndLog('git remote -v')
            runAndLog('git branch --show-current || true')
            runAndLog('git rev-parse HEAD')
            logMessage('Stage finished: Checkout')
        }

        withEnv(['CI=true', 'NODE_OPTIONS=--openssl-legacy-provider']) {
            stage('Build') {
                logMessage('Stage started: Build')
                runAndLog('npm install --no-audit --no-fund --prefer-offline')
                runAndLog('npm run build')
                logMessage('Stage finished: Build')
            }

            stage('Test') {
                logMessage('Stage started: Test')
                runAndLog('npm test -- --watchAll=false')
                logMessage('Stage finished: Test')
            }
        }
    } catch (err) {
        pipelineResult = 'FAILURE'
        logMessage("Pipeline failed: ${err}")
        throw err
    } finally {
        logMessage("Pipeline finished with status: ${pipelineResult}")
        archiveArtifacts artifacts: 'log.txt', allowEmptyArchive: true
    }
}
