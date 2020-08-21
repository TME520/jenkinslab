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
        stage('Check CFN params file before change') {
            steps {
                script {
                    def cfn_params_file = readFile(file: 'cloudformation/params/p7_default.json')
                    println(cfn_params_file)
                }
            }
        }
        stage('Use manually defined external IP') {
            when {
                beforeAgent true
                expression { params.authorized_ip != '' }
            }
            steps {
                script {
                    sh "sed -i \'s,SED028," + params.authorized_ip + ",g\' ./cloudformation/params/p7_default.json"
                }
            }
        }
        stage('Find my external IP') {
            when {
                beforeAgent true
                expression { params.authorized_ip == '' }
            }
            steps {
                script {
                    external_ip = sh(script: 'curl ifconfig.me', returnStdout: true)
                    println external_ip
                    sh "sed -i \'s,SED028," + external_ip + "/0,g\' ./cloudformation/params/p7_default.json"
                }
            }
        }
        stage('CFN params file generation') {
            environment {
                SLACK_TOKEN = credentials('slack_token')
                SUMO_ENDPOINT = credentials('sumo_endpoint')
                AWS_SECRET_KEY_DATA = credentials('aws_access_key')
                def (value1, value2) = AWS_SECRET_KEY_DATA.tokenize( ':' )
                println value1
            }
            steps {
                script {
                    echo "Customizing CFN params file..."
                    echo "AWS secret key data: " + AWS_SECRET_KEY_DATA
                    sh label: '', script: '''
                    sed -i \'s/SED001/''' + params.p7_instance_name + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED002/''' + params.p7_instance_client + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED003/''' + params.p7_instance_env + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED004/''' + params.p7_instance_project + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED005/''' + SLACK_TOKEN + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED006/''' + params.azure_storage_account_name + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED007/''' + params.azure_storage_account_key + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED008/''' + params.nt_api_user + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED009/''' + params.nt_api_password + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED010/''' + params.azure_devops_pat + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s,SED011,''' + SUMO_ENDPOINT + ''',g\' ./cloudformation/params/p7_default.json
                    sed -i \'s,SED012,http://''' + params.dynamodb_url + ''',g\' ./cloudformation/params/p7_default.json
                    sed -i \'s,SED013,''' + params.chatbotone_data_folder + ''',g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED014/''' + params.dashboard_filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED015/''' + params.advanced_dashboard_filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED016/''' + params.dashboard_base_url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED017/''' + params.azure_devops_url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s,SED018,''' + params.logs_folder + ''',g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED019/''' + params.log_filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED020/''' + params.config_filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED021/''' + params.nt_api_search_url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED022/''' + params.nt_api_login_url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED023/''' + params.enable_blinkstick + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED024/''' + params.enable_slack + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED025/''' + params.enable_sumologic + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED026/''' + params.enable_dashboard + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED027/''' + params.stack_name + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED029/''' + params.hosted_zone_name + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED030/''' + params.stack_name + '.' + params.dashboard_base_url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED031/''' + params.store_dash_azure + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED032/''' + params.store_dash_aws + '''/g\' ./cloudformation/params/p7_default.json
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
        stage('Check AWS CLI version') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--version") }
            }
        }
        stage('Save artifact: p7_default.json') {
            steps {
                archiveArtifacts artifacts: 'cloudformation/params/p7_default.json', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
            }
        }
        stage('Sync CFN params file with S3') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 s3 sync --exclude '*' --include 'p7_default.json' ${WORKSPACE}/cloudformation/params/ s3://cf-templates-w4ea9ebnhuyx-ap-southeast-2/") }
            }
        }
        stage('Start deployment of P7') {
            steps {
                build job: 'deployProtocol7', parameters: [string(name: 'stack_name', value: params.stack_name)], quietPeriod: 3
            }
        }
    }
}