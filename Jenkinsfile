pipeline {
    agent any

    environment {
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        CHANGE_DIR = 'src/database'      // Root folder for database scripts
        DEV_SCHEMA = 'dev_user_1'
        PROD_SCHEMA = 'prod_user_1'
        DEV_SERVICE = 'devadb_low'
        PROD_SERVICE = 'prodadb_low'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sn0313/cicd-adb.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Deploy to DEV') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: '3086a9e1-0196-41a4-ace4-12b7673da99b',
                            usernameVariable: 'DB_USER',
                            passwordVariable: 'DB_PSW'
                        ),
                        file(credentialsId: 'cicd-adb-wallet', variable: 'DEV_WALLET_FILE')
                    ]) {
                        sh """
                        # Create temporary folder for wallet
                        TMP_WALLET_DIR=\$(mktemp -d)
                        unzip -o \$DEV_WALLET_FILE -d \$TMP_WALLET_DIR

                        export TNS_ADMIN=\$TMP_WALLET_DIR

                        SCHEMA_PATH="${CHANGE_DIR}/${DEV_SCHEMA}/tables"

                        echo "Looking for DEV SQL files in: \$SCHEMA_PATH"

                        if [ -d "\$SCHEMA_PATH" ]; then
                            for sqlfile in \$SCHEMA_PATH/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running: \$sqlfile"

                                ${SQLCL} /nolog <<EOF
connect \$DB_USER/\$DB_PSW@${DEV_SERVICE}
@\${sqlfile}
exit;
EOF
                            done
                        else
                            echo "❌ No DEV folder found at \$SCHEMA_PATH"
                        fi
                        """
                    }
                }
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
                        file(credentialsId: 'cicd-prod-adb-wallet', variable: 'PROD_WALLET_FILE')
                    ]) {
                        sh """
                        TMP_WALLET_DIR=\$(mktemp -d)
                        unzip -o \$PROD_WALLET_FILE -d \$TMP_WALLET_DIR

                        export TNS_ADMIN=\$TMP_WALLET_DIR

                        SCHEMA_PATH="${CHANGE_DIR}/${PROD_SCHEMA}/tables"

                        echo "Looking for PROD SQL files in: \$SCHEMA_PATH"

                        if [ -d "\$SCHEMA_PATH" ]; then
                            for sqlfile in \$SCHEMA_PATH/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running: \$sqlfile"

                                ${SQLCL} /nolog <<EOF
connect \$DB_USER/\$DB_PSW@${PROD_SERVICE}
@\${sqlfile}
exit;
EOF
                            done
                        else
                            echo "❌ No PROD folder found at \$SCHEMA_PATH"
                        fi
                        """
                    }
                }
            }
        }
    }

    post {
        success { echo 'Deployment completed!' }
        failure { echo 'Deployment failed.' }
    }
}
