pipeline {
    agent any

    environment {
        APP_NAME   = "flask-sqlite-app"
        S3_BUCKET  = "flask-artifacts-bucket-group14"   
        REGION     = "us-east-1"
        DEPLOY_HOST = "flask"                        // host alias from your Ansible inventory
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature/Elvis14-versioning-and-rollback',
                    url: 'https://github.com/Elvis-Ikay/Flask-Sqlite-Application-group14.git'

            }
        }

        stage('Build Artifact') {
            steps {
                sh """
                VERSION=\$(date +%Y%m%d%H%M%S)-\$(git rev-parse --short HEAD)
                echo \$VERSION > version.txt

                mkdir -p build
                cp -r app.py init_db.py init_db.sql requirements.txt templates playbooks build/
                tar -czf ${APP_NAME}-\$VERSION.tar.gz -C build .
                """
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(region: "${REGION}", credentials: 'aws-creds') {
                    
                    s3Upload(
                        file: "${APP_NAME}-$VERSION.tar.gz",
                        bucket: "${S3_BUCKET}",
                        path: "artifacts/${APP_NAME}-$VERSION.tar.gz"
                    )
                }
            }
        }

        stage('Deploy with Ansible') {
            agent { label 'ansible-master' }
            steps {
                sh '''
                VERSION=$(cat version.txt)
                ansible-playbook playbooks/deploy.yml \
                  --extra-vars "app_name=${APP_NAME} app_version=$VERSION s3_bucket=${S3_BUCKET} region=${REGION}"
                '''
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
