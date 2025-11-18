pipeline {
    agent any

    environment {
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        PROD_SERVICE = 'cicdprodadb_low'
        ORDS_SCHEMA = 'DEV_USER_1'
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
                        # Prepare wallet
                        TMP_WALLET_DIR=\$(mktemp -d)
                        unzip -o \$PROD_WALLET_FILE -d \$TMP_WALLET_DIR
                        WALLET_SUBDIR=\$(find \$TMP_WALLET_DIR -mindepth 1 -maxdepth 1 -type d | head -n 1)
                        [ -z "\$WALLET_SUBDIR" ] && WALLET_SUBDIR=\$TMP_WALLET_DIR
                        export TNS_ADMIN=\$WALLET_SUBDIR

                        echo "Deploying all SQL files from dev_user_1 folder to PROD..."

                        # Collect SQL files
                        SQL_FILES=\$(find dist/releases/ords/dev_user_1/ -name '*.sql' | sort)
                        if [ -z "\$SQL_FILES" ]; then
                            echo "No SQL files found in dev_user_1 folder."
                            exit 0
                        fi

                        # Run deployment in SQLCL
                        ${SQLCL} /nolog <<EOF
connect \$DB_USER/\$DB_PSW@${PROD_SERVICE}

-- Disable ORDS schema
BEGIN
    ORDS.ENABLE_SCHEMA(p_enabled => FALSE, p_schema => '${ORDS_SCHEMA}');
END;
/

-- Run each SQL file
EOF

                        for sqlfile in \$SQL_FILES; do
                            echo "Running SQL file: \$sqlfile"
                            ${SQLCL} \$DB_USER/\$DB_PSW@${PROD_SERVICE} @"\$sqlfile"
                        done

                        ${SQLCL} /nolog <<EOF
connect \$DB_USER/\$DB_PSW@${PROD_SERVICE}

-- Re-enable ORDS schema
BEGIN
    ORDS.ENABLE_SCHEMA(p_enabled => TRUE, p_schema => '${ORDS_SCHEMA}');
END;
/
exit
EOF
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
