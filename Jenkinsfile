pipeline {
    agent any

    environment {
        DEV_USERNAME = credentials('3086a9e1-0196-41a4-ace4-12b7673da99b')
        PROD_USERNAME = credentials('cicd-prod-adb')
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
                    sh """
                    ${SQLCL} ${DEV_USERNAME}@\"cicdadb_low\" /nolog <<EOF
                    SET SQLFORMAT ANSICONSOLE
                    @./scripts/deploy_dev.sql
                    EXIT
                    EOF
                    """
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                input message: "Deploy to PROD? Confirm to continue..."
                script {
                    sh """
                    ${SQLCL} ${PROD_USERNAME}@\"cicdprodadb_low\" /nolog <<EOF
                    SET SQLFORMAT ANSICONSOLE
                    @./scripts/deploy_prod.sql
                    EXIT
                    EOF
                    """
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
