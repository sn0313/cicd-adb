pipeline {
    agent any

    environment {
        PROD_SERVICE = 'cicdprodadb_low' // TNS alias in your wallet
        CHANGELOG_FILE = 'changelog.xml'  // Your Liquibase changelog
        LIQUIBASE_CLASSPATH = 'ojdbc8.jar' // JDBC driver
    }

    triggers {
        pollSCM('H/5 * * * *') // Optional: poll GitHub every 5 minutes
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
            when {
                expression { currentBuild.result != 'SUCCESS' } // Run only if there are changes
            }
            steps {
                input message: "Deploy to PROD?"

                withCredentials([
                    usernamePassword(
                        credentialsId: 'prod-adb-creds', 
                        usernameVariable: 'DB_USER', 
                        passwordVariable: 'DB_PSW'
                    ),
                    file(
                        credentialsId: 'prod-adb-wallet', 
                        variable: 'PROD_WALLET_FILE'
                    )
                ]) {

                    script {
                        // Unzip wallet to temp directory
                        def tmpWalletDir = sh(script: 'mktemp -d', returnStdout: true).trim()
                        sh "unzip -o $PROD_WALLET_FILE -d $tmpWalletDir"

                        // Set TNS_ADMIN to the wallet folder
                        env.TNS_ADMIN = tmpWalletDir
                        
                        echo "Running Liquibase deployment..."
                        sh """
                        liquibase \
                            --url=jdbc:oracle:thin:@${PROD_SERVICE} \
                            --username=$DB_USER \
                            --password=$DB_PSW \
                            --changeLogFile=${CHANGELOG_FILE} \
                            --classpath=${LIQUIBASE_CLASSPATH} \
                            update
                        """
                    }
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed.' }
        always {
            echo 'Cleaning up workspace...'
            deleteDir()
        }
    }
}
