pipeline {
    agent any

    environment {
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        PROD_SERVICE = 'cicdprodadb_low'
        DEV_SCHEMA_FOLDER = 'src/database/dev_user_1'
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

                        echo "Detecting SQL changes in the dev_user_1 folder..."

                        # Detect all changed SQL files in the dev_user_1 folder since last commit
                        CHANGED_FILES=\$(git diff --name-only origin/main..HEAD | grep '^${DEV_SCHEMA_FOLDER}/' | grep -E '\\.sql\$' || true)

                        if [ -z "\$CHANGED_FILES" ]; then
                            echo "No SQL changes detected in ${DEV_SCHEMA_FOLDER}."
                        else
                            for sqlfile in \$CHANGED_FILES; do
                                echo "Deploying SQL file: \$sqlfile"

                                ${SQLCL} /nolog <<EOF
                                connect \$DB_USER/\$DB_PSW@${PROD_SERVICE}

                                BEGIN
                                    EXECUTE IMMEDIATE q'[
                                        @\$sqlfile
                                    ]';
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
                        fi
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
