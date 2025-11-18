pipeline {
    agent any

    environment {
        DEV_USERNAME = credentials('3086a9e1-0196-41a4-ace4-12b7673da99b') // DEV ADB creds
        PROD_USERNAME = credentials('cicd-prod-adb')                        // PROD ADB creds
        DEV_WALLET = '/var/jenkins_home/wallets/cicd-adb-wallet'
        PROD_WALLET = '/var/jenkins_home/wallets/cicd-prod-adb-wallet'
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
        BRANCH_NAME = 'test-1.0' // branch for release
        SCHEMA_DEV = 'dev_user_1'
        SCHEMA_PROD = 'prod_user_1'
        CHANGE_DIR = "dist/releases/next/changes/${BRANCH_NAME}"
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
                    // Mask credentials when running shell
                    withCredentials([usernamePassword(credentialsId: '3086a9e1-0196-41a4-ace4-12b7673da99b',
                                                     usernameVariable: 'DB_USER',
                                                     passwordVariable: 'DB_PSW')]) {

                        sh """
                        for sqlfile in ${CHANGE_DIR}/${SCHEMA_DEV}/tables/*.sql; do
                            ${SQLCL} \$DB_USER/\$DB_PSW@\"cicdadb_low\" /nolog <<EOF
                            SET SQLFORMAT ANSICONSOLE
                            @\$sqlfile
                            EXIT
EOF
                        done
                        """
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                input message: "Deploy to PROD? Confirm to continue..."
                script {
                    // Mask credentials when running shell
                    withCredentials([usernamePassword(credentialsId: 'cicd-prod-adb',
                                                     usernameVariable: 'DB_USER',
                                                     passwordVariable: 'DB_PSW')]) {

                        sh """
                        for sqlfile in ${CHANGE_DIR}/${SCHEMA_PROD}/tables/*.sql; do
                            ${SQLCL} \$DB_USER/\$DB_PSW@\"cicdprodadb_low\" /nolog <<EOF
                            SET SQLFORMAT ANSICONSOLE
                            @\$sqlfile
                            EXIT
EOF
                        done
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
