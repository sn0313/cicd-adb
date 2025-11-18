pipeline {
    agent any

    environment {
        PROD_SERVICE = 'cicdprodadb_low' // TNS alias in your wallet
        CHANGELOG_FILE = 'changelog.xml'  // Your Liquibase changelog
        LIQUIBASE_CLASSPATH = 'ojdbc8.jar' // JAR uploaded to repo root
    }

    triggers {
        // Optional: poll GitHub every 5 minutes
        pollSCM('H/5 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sn0313/cicd-adb.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Detect Changes') {
            steps {
                script {
                    // Get previous and current commits
                    def prevCommit = sh(script: "git rev-parse HEAD~1", returnStdout: true).trim()
                    def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    echo "Previous commit: ${prevCommit}"
                    echo "Current commit: ${currentCommit}"

                    // Detect changes in dev_user_1 folder
                    def changedFiles = sh(
                        script: "git diff --name-only ${prevCommit} ${currentCommit} | grep '^dist/releases/ords/dev_user_1/' || true",
                        returnStdout: true
                    ).trim()

                    if (!changedFiles) {
                        echo "No changes detected in dev_user_1 folder. Skipping deployment."
                        currentBuild.result = 'SUCCESS'
                        return
                    } else {
                        echo "Changed SQL files:\n${changedFiles}"
                    }
                }
            }
        }

        stage('Deploy to PROD with Liquibase') {
            steps {
                input message: "Deploy to PROD?" // Optional manual approval

                withCredentials([
                    usernamePassword(
                        credentialsId: 'cicd-prod-adb', 
                        usernameVariable: 'DB_USER', 
                        passwordVariable: 'DB_PSW'
                    ),
                    file(
                        credentialsId: 'cicd-prod-adb-wallet', 
                        variable: 'PROD_WALLET_FILE'
                    )
                ]) {

                    sh """
                    # Prepare Oracle Wallet
                    TMP_WALLET_DIR=\$(mktemp -d)
                    unzip -o \$PROD_WALLET_FILE -d \$TMP_WALLET_DIR
                    WALLET_SUBDIR=\$(find \$TMP_WALLET_DIR -mindepth 1 -maxdepth 1 -type d | head -n 1)
                    [ -z "\$WALLET_SUBDIR" ] && WALLET_SUBDIR=\$TMP_WALLET_DIR
                    export TNS_ADMIN=\$WALLET_SUBDIR

                    echo "Running Liquibase deployment..."
                    liquibase \
                      --url=jdbc:oracle:thin:@${PROD_SERVICE}?TNS_ADMIN=\$TNS_ADMIN \
                      --username=\$DB_USER \
                      --password=\$DB_PSW \
                      --changeLogFile=${CHANGELOG_FILE} \
                      --classpath=${LIQUIBASE_CLASSPATH} \
                      update
                    """
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed.' }
    }
}
