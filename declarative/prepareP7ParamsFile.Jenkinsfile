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
                expression { params.authorized-ip != '' }
            }
            steps {
                script {
                    sh "sed -i \'s,SED028," + params.authorized-ip + ",g\' ./cloudformation/params/p7_default.json"
                }
            }
        }
        stage('Find my external IP') {
            when {
                beforeAgent true
                expression { params.authorized-ip == '' }
            }
            steps {
                script {
                    external-ip = sh(script: 'curl ifconfig.me', returnStdout: true)
                    println external-ip
                    sh "sed -i \'s,SED028," + external-ip + ",g\' ./cloudformation/params/p7_default.json"
                }
            }
        }
        stage('CFN params file generation') {
            steps {
                script {
                    echo "Customizing CFN params file..."
                    sh label: '', script: '''
                    sed -i \'s/SED001/''' + params.p7-instance-name + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED002/''' + params.p7-instance-client + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED003/''' + params.p7-instance-env + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED004/''' + params.p7-instance-project + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED005/''' + params.slack-token + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED006/''' + params.azure-storage-account-name + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED007/''' + params.azure-storage-account-key + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED008/''' + params.nt-api-user + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED009/''' + params.nt-api-password + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED010/''' + params.azure-devops-pat + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED011/''' + params.sumo-endpoint + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s,SED012,http://''' + params.dynamodb-url + ''',g\' ./cloudformation/params/p7_default.json
                    sed -i \'s,SED013,''' + params.chatbotone-data-folder + ''',g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED014/''' + params.dashboard-filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED015/''' + params.advanced-dashboard-filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED016/''' + params.dashboard-base-url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED017/''' + params.azure-devops-url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s,SED018,''' + params.logs-folder + ''',g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED019/''' + params.log-filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED020/''' + params.config-filename + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED021/''' + params.nt-api-search-url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED022/''' + params.nt-api-login-url + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED023/''' + params.enable-blinkstick + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED024/''' + params.enable-slack + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED025/''' + params.enable-sumologic + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED026/''' + params.enable-dashboard + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED027/''' + params.stack-name + '''/g\' ./cloudformation/params/p7_default.json
                    sed -i \'s/SED029/''' + params.hosted-zone-name + '''/g\' ./cloudformation/params/p7_default.json
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
    }
}