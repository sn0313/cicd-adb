pipeline {
    agent any
    environment {
        TNS_ADMIN = '/tmp/tmp_wallet_dir' // will be set dynamically
        PROD_USER = 'Prod_User_1'
        PROD_DB = 'cicdprodadb_low'
        PROD_PSW = credentials('prod-db-password') // Jenkins credential
        PROD_WALLET_FILE = credentials('prod-wallet-file') // Jenkins credential
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Deploy to PROD') {
            steps {
                input "Deploy to PROD?"
                script {
                    // Unzip wallet
                    sh '''
                        TMP_WALLET_DIR=$(mktemp -d)
                        unzip -o $PROD_WALLET_FILE -d $TMP_WALLET_DIR
                        export TNS_ADMIN=$TMP_WALLET_DIR
                    '''

                    // Find all SQL files in dev_user_1
                    def sqlFiles = sh(script: "find dist/releases/ords/dev_user_1/ -name '*.sql' | sort", returnStdout: true).trim()

                    if (sqlFiles) {
                        echo "Deploying SQL files:\n${sqlFiles}"

                        // Build a SQL script to disable schema, run all SQLs, then enable schema
                        def deploymentScript = sqlFiles.split("\n").collect { file ->
                            return "@${file}"
                        }.join('\n')

                        writeFile file: 'deploy_all.sql', text: """
                        BEGIN
                            -- Disable ORDS schema first
                            ORDS.ENABLE_SCHEMA(
                                p_enabled => FALSE,
                                p_schema  => '${PROD_USER}',
                                p_url_mapping_type => 'BASE_PATH'
                            );
                        END;
                        /
                        ${deploymentScript}
                        BEGIN
                            -- Re-enable ORDS schema
                            ORDS.ENABLE_SCHEMA(
                                p_enabled => TRUE,
                                p_schema  => '${PROD_USER}',
                                p_url_mapping_type => 'BASE_PATH'
                            );
                        END;
                        /
                        """

                        // Execute SQLCL
                        sh """
                        /opt/oracle/sqlcl/bin/sql ${PROD_USER}/${PROD_PSW}@${PROD_DB} @deploy_all.sql
                        """
                    } else {
                        echo "No SQL files found to deploy."
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'PROD deployment stage finished.'
        }
    }
}
