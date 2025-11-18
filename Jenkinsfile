pipeline {
    agent any

    environment {
        LIQUIBASE_CLASSPATH = 'ojdbc8.jar'   // Make sure ojdbc8.jar is in repo root
        CHANGELOG_FILE = 'changelog.xml'
        DB_USER = 'Prod_User_1'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Detect Changes') {
            steps {
                script {
                    // Get previous and current commit
                    PREV_COMMIT = sh(script: 'git rev-parse HEAD~1', returnStdout: true).trim()
                    CURRENT_COMMIT = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()

                    echo "Previous commit: ${PREV_COMMIT}"
                    echo "Current commit: ${CURRENT_COMMIT}"

                    // Check for changes in dev_user_1 folder
                    CHANGED_FILES = sh(
                        script: "git diff --name-only ${PREV_COMMIT} ${CURRENT_COMMIT} | grep '^dist/releases/ords/dev_user_1/' || true",
                        returnStdout: true
                    ).trim()

                    if (CHANGED_FILES == '') {
                        echo "No changes detected in dev_user_1 folder. Skipping deployment."
                        currentBuild.result = 'SUCCESS'
                        return
                    } else {
                        echo "Detected changes in dev_user_1 folder. Proceeding with deployment."
                    }
                }
            }
        }

        stage('Deploy to PROD with Liquibase') {
            when {
                expression { CHANGED_FILES != '' } // Only run if there are changes
            }
            steps {
                input message: 'Deploy to PROD?', ok: 'Deploy'

                withCredentials([
                    string(credentialsId: 'DB_PSW', variable: 'DB_PSW'),
                    file(credentialsId: 'PROD_WALLET_FILE', variable: 'PROD_WALLET_FILE')
                ]) {
                    script {
                        // Unzip wallet to temp directory
                        TMP_WALLET_DIR = sh(script: 'mktemp -d', returnStdout: true).trim()
                        sh "unzip -o $PROD_WALLET_FILE -d $TMP_WALLET_DIR"

                        // Find wallet subdir if any
                        WALLET_SUBDIR = sh(script: "find $TMP_WALLET_DIR -mindepth 1 -maxdepth 1 -type d | head -n 1", returnStdout: true).trim()
                        if (WALLET_SUBDIR == '') {
                            WALLET_SUBDIR = TMP_WALLET_DIR
                        }

                        // Export TNS_ADMIN
                        env.TNS_ADMIN = WALLET_SUBDIR

                        echo "Running Liquibase deployment with SSO wallet..."
                        sh """
                            liquibase \
                                --url=jdbc:oracle:thin:@cicdprodadb_low \
                                --username=$DB_USER \
                                --password=$DB_PSW \
                                --changeLogFile=$CHANGELOG_FILE \
                                --classpath=$LIQUIBASE_CLASSPATH \
                                -Doracle.net.wallet_location=$TNS_ADMIN \
                                update
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
        always {
            echo 'Cleaning up workspace...'
            deleteDir()
        }
    }
}
