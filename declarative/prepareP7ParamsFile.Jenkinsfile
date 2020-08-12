@Library('github.com/releaseworks/jenkinslib') _

pipeline {
    agent any
    environment {
        p7_instance_name="${params.p7_instance_name}"
        p7_instance_client="${params.p7_instance_client}"
        p7_instance_env="${params.p7_instance_env}"
        p7_instance_project="${params.p7_instance_project}"
    }
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
                    sed -i \'s/SED001/$p7_instance_name/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED002/$p7_instance_client/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED003/$p7_instance_env/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED004/$p7_instance_project/g\' ./cloudformation/params/p7_default.json
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
        stage('Sync CFN params file with S3') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 s3 sync --exclude '*' --exclude 'params' --include '*.json' ${WORKSPACE}/cloudformation/ s3://cf-templates-w4ea9ebnhuyx-ap-southeast-2/") }
            }
        }
    }
}