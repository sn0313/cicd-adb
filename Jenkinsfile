pipeline {
    agent any

    environment {
        DEV_WALLET = '/var/jenkins_home/wallets/cicd-adb-wallet'
        PROD_WALLET = '/var/jenkins_home/wallets/cicd-prod-adb-wallet'
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        CHANGE_DIR = 'dist/releases/next/changes'
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
                    withCredentials([usernamePassword(
                        credentialsId: '3086a9e1-0196-41a4-ace4-12b7673da99b',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PSW'
                    )]) {

                        sh """
                        export TNS_ADMIN=${DEV_WALLET}

                        SCHEMA_PATH="${CHANGE_DIR}/dev/${DEV_SCHEMA}/tables"

                        if [ -d "\$SCHEMA_PATH" ]; then
                            for sqlfile in \$SCHEMA_PATH/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running \$sqlfile ..."

                                ${SQLCL} /nolog <<EOF
                                connect \$DB_USER/\$DB_PSW@${DEV_SERVICE}
                                @\$sqlfile
                                exit;
EOF

                            done
                        else
                            echo "No DEV changes found in \$SCHEMA_PATH"
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
                    withCredentials([usernamePassword(
                        credentialsId: 'cicd-prod-adb',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PSW'
                    )]) {

                        sh """
                        export TNS_ADMIN=${PROD_WALLET}

                        SCHEMA_PATH="${CHANGE_DIR}/prod/${PROD_SCHEMA}/tables"

                        if [ -d "\$SCHEMA_PATH" ]; then
                            for sqlfile in \$SCHEMA_PATH/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running \$sqlfile ..."

                                ${SQLCL} /nolog <<EOF
                                connect \$DB_USER/\$DB_PSW@${PROD_SERVICE}
                                @\$sqlfile
                                exit;
EOF

                            done
                        else
                            echo "No PROD changes found in \$SCHEMA_PATH"
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

