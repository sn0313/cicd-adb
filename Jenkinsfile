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
                        TMP_WALLET_DIR=\$(mktemp -d)
                        unzip -o \$PROD_WALLET_FILE -d \$TMP_WALLET_DIR
                        WALLET_SUBDIR=\$(find \$TMP_WALLET_DIR -mindepth 1 -maxdepth 1 -type d | head -n 1)
                        [ -z "\$WALLET_SUBDIR" ] && WALLET_SUBDIR=\$TMP_WALLET_DIR
                        export TNS_ADMIN=\$WALLET_SUBDIR

                        echo "Deploying all SQL files from dev_user_1 folder to PROD..."

                        for sqlfile in dist/releases/ords/dev_user_1/*.sql; do
                            [ -f "\$sqlfile" ] || continue
                            echo "Running SQL file: \$sqlfile"

                            ${SQLCL} /nolog <<EOF
                            connect \$DB_USER/\$DB_PSW@${PROD_SERVICE}
                            BEGIN
                                EXECUTE IMMEDIATE ' ' || REPLACE(
                                    REPLACE(REPLACE(
                                    REPLACE(REPLACE(
                                    REPLACE(@\$sqlfile, '\\n', ' '), '\\r', ' '), '"', '""'), '''', '\'''), ';', ''), '/', '');
                            EXCEPTION
                                WHEN OTHERS THEN
                                    IF SQLCODE != -955 THEN -- ignore 'table already exists'
                                        RAISE;
                                    END IF;
                            END;
                            /
                            exit;
EOF
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

