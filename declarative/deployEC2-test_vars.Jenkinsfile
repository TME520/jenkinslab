@Library('github.com/releaseworks/jenkinslib') _

pipeline {
    agent any
    stages {
        stage('Cleanup') {
            steps {
                deleteDir()
            }
        }
        stage('Retrieve CFN template') {
            steps {
                dir('cloudformation') {
                    git changelog: false, credentialsId: 'c0d66b24-e928-45f3-8da2-0b3f960ca800', poll: false, url: 'https://github.com/TME520/cloudformation.git'
                    echo "GIT clone success check: all right !"
                }
            }
        }
        stage('Sync CFN template with S3') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 s3 sync --exclude '*' --include '*.json' ${WORKSPACE}/cloudformation/ s3://cf-templates-w4ea9ebnhuyx-ap-southeast-2/") }
            }
        }
        stage('Create stack') {
            steps {
                echo "Creating the stack"
                echo "Stack name: ${params.stack_name}"
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 cloudformation create-stack --stack-name ${params.stack_name} --template-url https://cf-templates-w4ea9ebnhuyx-ap-southeast-2.s3-ap-southeast-2.amazonaws.com/singleEC2-test_vars.json --parameters ParameterKey=ChosenDemon,ParameterValue=${params.ChosenDemon}") }
            }
        }
        stage('Monitor stack creation') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 cloudformation wait stack-create-complete --stack-name ${params.stack_name}") }
            }
        }
        stage('Clean workspace') {
            steps {
                cleanWs notFailBuild: true
            }
        }
    }
}