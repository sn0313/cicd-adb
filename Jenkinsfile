pipeline {
    agent any

    environment {
        PROD_SERVICE = 'cicdprodadb_low'
        LIQUIBASE_CLASSPATH = 'ojdbc8.jar' // jar is at root of repo
        CHANGELOG_FILE = 'changelog.xml'   // Your Liquibase changelog in repo
    }

    triggers {
        // Poll GitHub for changes every 5 minutes
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
                withCredentials([usernamePassword(
                    credentialsId: 'cicd-prod-adb',
                    usernameVariable: 'DB_USER',
                    passwordVariable: 'DB_PSW'
                )]) {
                    sh """
                        echo "Running Liquibase deployment..."
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

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed.' }
    }
}

