pipeline {
    agent any

    environment {
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        PROD_SERVICE = 'cicdprodadb_low'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sn0313/cicd-adb.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Deploy to PROD') {
            steps {
                input message: "Deploy to PROD?"
                script {
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
                        # Create temporary wallet directory
                        TMP_WALLET_DIR=\$(mktemp -d)
                        unzip -o \$PROD_WALLET_FILE -d \$TMP_WALLET_DIR
                        WALLET_SUBDIR=\$(find \$TMP_WALLET_DIR -mindepth 1 -maxdepth 1 -type d | head -n 1)
                        [ -z "\$WALLET_SUBDIR" ] && WALLET_SUBDIR=\$TMP_WALLET_DIR
                        export TNS_ADMIN=\$WALLET_SUBDIR

                        echo "Deploying all SQL files from dev_user_1 folder to PROD..."

                        # Loop through all SQL files in dev_user_1 folder
                        for sqlfile in dist/releases/ords/dev_user_1/*.sql; do
                            if [ -f "\$sqlfile" ]; then
                                echo "Running SQL file: \$sqlfile"
                                ${SQLCL} \$DB_USER/\$DB_PSW@${PROD_SERVICE} @\$sqlfile
                            fi
                        done
                        """
                    }
                }
            }
        }
    }

    post {
        success { echo 'PROD deployment completed!' }
        failure { echo 'PROD deployment failed.' }
    }
}
