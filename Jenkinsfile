pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/adityasatyam12/ashrams.git'
            }
        }

        stage('Terraform Init') {
            steps {
                bat 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                bat 'terraform validate'
            }
        }

        stage('Security Scan') {
            steps {
                bat 'terraform fmt -check'
            }
        }

        stage('Archive Artifact') {
            steps {
                bat 'zip -r artifact.zip *'
                archiveArtifacts artifacts: 'artifact.zip'
            }
        }
    }
}