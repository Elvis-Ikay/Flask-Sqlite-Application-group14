pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        APP_NAME   = "flask-sqlite-app"
        S3_BUCKET  = "flask-artifacts-bucket-group14"   
        REGION     = "us-east-1"
        DEPLOY_HOST = "flask"                        
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature/Elvis14-jenkins-setup',
                    url: 'https://github.com/Elvis-Ikay/Flask-Sqlite-Application-group14.git'

            }
        }

        stage('Build Artifact') {
            steps {
                sh """
                mkdir -p build
                cp -r app.py init_db.py init_db.sql requirements.txt templates playbooks build/
                # Package everything in build/ into one tar.gz
                tar -czf ${APP_NAME}.tar.gz -C build .
                """
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(region: "${REGION}", credentials: 'aws-creds') {
                    s3Upload(
                        file: "${APP_NAME}.tar.gz",
                        bucket: "${S3_BUCKET}",
                        path: "artifacts/${APP_NAME}.tar.gz"
                    )
                }
            }
        }

        stage('Deploy with Ansible') {
            agent { label 'ansible-master' }
            steps {
                    sh """
                    ansible-playbook  playbooks/deploy.yml \
                    --extra-vars "app_name=${APP_NAME} s3_bucket=${S3_BUCKET} region=${REGION}"
                    """
            }
        }
    }

    post {
        success {
            echo "✅ Build, Upload, and Deploy completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs."
        }
    }
}
