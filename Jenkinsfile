pipeline {
    agent any

    environment {
        DEV_USERNAME = credentials('3086a9e1-0196-41a4-ace4-12b7673da99b')  // DEV DB credential
        PROD_USERNAME = credentials('cicd-prod-adb')                        // PROD DB credential
        DEV_WALLET = '/var/jenkins_home/wallets/cicd-adb-wallet'
        PROD_WALLET = '/var/jenkins_home/wallets/cicd-prod-adb-wallet'
        SQLCL = '/opt/oracle/sqlcl/bin/sql'
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
                    // Mask credentials properly
                    withCredentials([usernamePassword(credentialsId: '3086a9e1-0196-41a4-ace4-12b7673da99b', 
                                                     usernameVariable: 'DB_USER', 
                                                     passwordVariable: 'DB_PSW')]) {
                        sh """
                        # Run all SQL scripts in the 'scripts' folder
                        for sqlfile in \$(ls ./scripts/*.sql | sort); do
                            ${SQLCL} \$DB_USER/\$DB_PSW@cicdadb_low /nolog <<EOF
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
                    withCredentials([usernamePassword(credentialsId: 'cicd-prod-adb', 
                                                     usernameVariable: 'DB_USER', 
                                                     passwordVariable: 'DB_PSW')]) {
                        sh """
                        # Run all SQL scripts in the 'scripts' folder
                        for sqlfile in \$(ls ./scripts/*.sql | sort); do
                            ${SQLCL} \$DB_USER/\$DB_PSW@cicdprodadb_low /nolog <<EOF
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
