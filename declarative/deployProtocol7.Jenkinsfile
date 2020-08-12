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
        stage('CFN template validation') {
            steps {
                echo "Validating the CloudFormation template..."
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 cloudformation validate-template --template-url https://cf-templates-w4ea9ebnhuyx-ap-southeast-2.s3-ap-southeast-2.amazonaws.com/protocol7.json") }
                echo "...done with validation."
            }
        }
        stage('Check CFN params file before change') {
            steps {
                script {
                    def cfn_params_file = readFile(file: 'cloudformation/params/p7_default.json')
                    println(cfn_params_file)
                }
            }
        }
        stage('CFN params file generation') {
            steps {
                script {
                    echo "Customizing CFN params file..."
                    sh label: '', script: '''
                    sed -i \'s/SED001/${params.p7_instance_name}/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED002/${params.p7_instance_client}/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED003/${params.p7_instance_env}/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED004/${params.p7_instance_project}/g\' ./cloudformation/params/p7_default.json
                    '''
                }
            }
        }
        stage('Check CFN params file after change') {
            steps {
                script {
                    def cfn_params_file = readFile(file: 'cloudformation/params/p7_default.json')
                    println(cfn_params_file)
                }
            }
        }
        stage('Create stack') {
            steps {
                echo "Creating the stack"
                echo "Stack name: ${params.stack_name}"
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 cloudformation create-stack --stack-name ${params.stack_name} --template-url https://cf-templates-w4ea9ebnhuyx-ap-southeast-2.s3-ap-southeast-2.amazonaws.com/protocol7.json --parameters ParameterKey=HostedZoneName,ParameterValue=${params.hosted_zone_name} ParameterKey=SSHLocation,ParameterValue=${params.authorized_ip} ParameterKey=P7InstanceName,ParameterValue=${params.p7_instance_name} ParameterKey=P7InstanceClient,ParameterValue=${params.p7_instance_client} ParameterKey=P7InstanceEnv,ParameterValue=${params.p7_instance_env} ParameterKey=P7InstanceProject,ParameterValue=${params.p7_instance_project}") }
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
    post {
        failure {
            echo "last_started: $last_started"
        }
    }
}