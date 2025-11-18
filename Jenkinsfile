pipeline {
    agent any

    environment {
        DEV_WALLET = '/var/jenkins_home/wallets/cicd-adb-wallet'
        PROD_WALLET = '/var/jenkins_home/wallets/cicd-prod-adb-wallet'
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        CHANGE_DIR = 'dist/releases/next/changes'
        SCHEMA_DEV = 'test-1.0/dev_user_1'
        SCHEMA_PROD = 'test-1.0/prod_user_1'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sn0313/cicd-adb.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    // Use Jenkins credentials for dev DB
                    withCredentials([usernamePassword(credentialsId: '3086a9e1-0196-41a4-ace4-12b7673da99b',
                                                     usernameVariable: 'DB_USER',
                                                     passwordVariable: 'DB_PSW')]) {
                        sh """
                        if [ -d "${CHANGE_DIR}/${SCHEMA_DEV}/tables" ]; then
                            for sqlfile in ${CHANGE_DIR}/${SCHEMA_DEV}/tables/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running \$sqlfile ..."
                                ${SQLCL} \$DB_USER/\$DB_PSW@\"cicdadb_low\" /nolog @\$sqlfile
                            done
                        else
                            echo "No changes found for DEV in ${CHANGE_DIR}/${SCHEMA_DEV}/tables"
                        fi
                        """
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                input message: "Deploy to PROD? Confirm to continue..."
                script {
                    // Use Jenkins credentials for prod DB
                    withCredentials([usernamePassword(credentialsId: 'cicd-prod-adb',
                                                     usernameVariable: 'DB_USER',
                                                     passwordVariable: 'DB_PSW')]) {
                        sh """
                        if [ -d "${CHANGE_DIR}/${SCHEMA_PROD}/tables" ]; then
                            for sqlfile in ${CHANGE_DIR}/${SCHEMA_PROD}/tables/*.sql; do
                                [ -f "\$sqlfile" ] || continue
                                echo "Running \$sqlfile ..."
                                ${SQLCL} \$DB_USER/\$DB_PSW@\"cicdprodadb_low\" /nolog @\$sqlfile
                            done
                        else
                            echo "No changes found for PROD in ${CHANGE_DIR}/${SCHEMA_PROD}/tables"
                        fi
                        """
                    }
                }
            }
        }

    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
