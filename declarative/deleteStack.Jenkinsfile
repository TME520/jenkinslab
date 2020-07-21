@Library('github.com/releaseworks/jenkinslib') _

pipeline {
    agent any
    stages {
        stage('Cleanup') {
            steps {
                deleteDir()
            }
        }
        stage('Create stack') {
            steps {
                echo "Deleting the stack"
                echo "Stack name: ${params.stack_name}"
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 cloudformation create-stack --stack-name ${params.stack_name} --template-url https://cf-templates-w4ea9ebnhuyx-ap-southeast-2.s3-ap-southeast-2.amazonaws.com/EC2andR53.json --parameters ParameterKey=HostedZoneName,ParameterValue=${params.hosted_zone_name}") }
            }
        }
        stage('Monitor stack deletion') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 cloudformation wait stack-delete-complete --stack-name ${params.stack_name}") }
            }
        }
        stage('Clean workspace') {
            steps {
                cleanWs notFailBuild: true
            }
        }
    }
}