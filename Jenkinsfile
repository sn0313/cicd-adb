pipeline {
    agent any

    environment {
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        CHANGE_DIR = 'src/database'      // Root path for SQL scripts
        DEV_SCHEMA = 'dev_user_1'
        PROD_SCHEMA = 'prod_user_1'
        DEV_SERVICE = 'cicdadb_low'
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

        stage('Deploy to DEV') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: '3086a9e1-0196-41a4-ace4-12b7673da99b',
                            usernameVariable: 'DB_USER',
                            passwordVariable: 'DB_PSW'
                        ),
                        file(
                            credentialsId: 'cicd-adb-wallet',
                            variable: 'DEV_WALLET_FILE'
                        )
                    ]) {

                        sh """
                        # Create temp dir for wallet
                        TMP_WALLET_DIR=\$(mktemp -d)
                        unzip -o \$DEV_WALLET_FILE -d \$TMP_WALLET_DIR

                        # Find the actual wallet folder inside the ZIP
                        WALLET_SUBDIR=\$(find \$TMP_WALLET_DIR -mindepth 1 -maxdepth 1 -type d | head -n 1)
                        export TNS_ADMIN=\$WALLET_SUBDIR

                        SCHEMA_PATH="${CHANGE_DIR}/${DEV_SCHEMA}/tables"

                        echo "Looking for DEV SQL files in: \$SCHEMA_PATH"

                        if [ -d "\$SCHEMA_PATH" ]; then
                            for sqlfile in \$SCHEMA_PATH/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running: \$sqlfile"

                                ${SQLCL} /nolog <<EOF
                                connect \$DB_USER/\$DB_PSW@${DEV_SERVICE}
                                @\$sqlfile
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
                        file(
                            credentialsId: 'cicd-prod-adb-wallet',
                            variable: 'PROD_WALLET_FILE'
                        )
                    ]) {

                        sh """
                        TMP_WALLET_DIR=\$(mktemp -d)
                        unzip -o \$PROD_WALLET_FILE -d \$TMP_WALLET_DIR

                        # Fix: Use TMP_WALLET_DIR directly if there is no subfolder
                        WALLET_SUBDIR=\$(find \$TMP_WALLET_DIR -mindepth 1 -maxdepth 1 -type d | head -n 1)
                        if [ -z "\$WALLET_SUBDIR" ]; then
                            WALLET_SUBDIR=\$TMP_WALLET_DIR
                        fi
                        export TNS_ADMIN=\$WALLET_SUBDIR

                        SCHEMA_PATH="${CHANGE_DIR}/${PROD_SCHEMA}/tables"

                        echo "Looking for PROD SQL files in: \$SCHEMA_PATH"

                        if [ -d "\$SCHEMA_PATH" ]; then
                            for sqlfile in \$SCHEMA_PATH/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running: \$sqlfile"

                                ${SQLCL} /nolog <<EOF
                                connect \$DB_USER/\$DB_PSW@${PROD_SERVICE}
                                @\$sqlfile
                                exit;
EOF
                            done
                        else
                            echo "No PROD folder found at \$SCHEMA_PATH"
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
