pipeline {
    agent any

    environment {
        PROD_SERVICE = 'cicdprodadb_low'
        LIQUIBASE_CLASSPATH = '/opt/oracle/ojdbc8.jar'
        CHANGELOG_FILE = 'changelog.xml' // Your Liquibase changelog in repo
    }

    triggers {
        // Poll GitHub for changes (alternatively, configure webhook in GitHub)
        pollSCM('H/5 * * * *') // every 5 minutes
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
                    // Compare current HEAD with previous main commit
                    PREV_COMMIT = sh(script: "git rev-parse HEAD~1", returnStdout: true).trim()
                    CURRENT_COMMIT = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    echo "Previous commit: ${PREV_COMMIT}"
                    echo "Current commit: ${CURRENT_COMMIT}"

                    // Detect files changed in dev_user_1 folder
                    CHANGED_FILES = sh(
                        script: "git diff --name-only ${PREV_COMMIT} ${CURRENT_COMMIT} | grep '^dist/releases/ords/dev_user_1/' || true",
                        returnStdout: true
                    ).trim()

                    if (!CHANGED_FILES) {
                        echo "No changes detected in dev_user_1 folder. Skipping deployment."
                        currentBuild.result = 'SUCCESS'
                        error("No relevant changes")
                    } else {
                        echo "Changed SQL files:\n${CHANGED_FILES}"
                    }
                }
            }
        }

        stage('Deploy to PROD with Liquibase') {
            steps {
                input message: "Deploy to PROD?" // optional manual approval
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
        success { echo 'PROD deployment completed successfully!' }
        failure { echo 'PROD deployment failed.' }
    }
}
