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
                dir('fullstack-tutorial') {
                    git changelog: false, credentialsId: 'c0d66b24-e928-45f3-8da2-0b3f960ca800', poll: false, url: 'https://github.com/TME520/fullstack-tutorial.git'
                    echo "GIT clone success check: all right !"
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
                    external_ip = params.authorized_ip
                    println external_ip
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
                    external_ip = external_ip + '/32'
                    println external_ip
                }
            }
        }
        stage('Sync CFN template with S3') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 s3 sync --exclude '*' --include '*.json' ${WORKSPACE}/fullstack-tutorial/cfn/ s3://cf-templates-w4ea9ebnhuyx-ap-southeast-2/") }
            }
        }
        stage('Create stack') {
            environment {
                AWS_SECRET_KEY_DATA = credentials('aws_access_key')
            }
            steps {
                script {
                    def (aws_access_key_id, aws_secret_access_key) = AWS_SECRET_KEY_DATA.tokenize( ':' )
                    println aws_access_key_id
                    println aws_secret_access_key
                }
                echo "Creating the stack"
                echo "Stack name: ${params.stack_name}"
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '411e79d0-00f9-4be4-babb-c26fac151e88', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) { AWS("--region=ap-southeast-2 cloudformation create-stack --stack-name ${params.stack_name} --template-url https://cf-templates-w4ea9ebnhuyx-ap-southeast-2.s3-ap-southeast-2.amazonaws.com/orchestrated.json --parameters ParameterKey=HostedZoneName,ParameterValue=${params.hosted_zone_name} ParameterKey=SSHLocation,ParameterValue=${external_ip} ParameterKey=S3BucketName,ParameterValue=${params.s3_bucket} ParameterKey=AWSSecretKeyID,ParameterValue=${aws_access_key_id} ParameterKey=AWSSecretAccessKey,ParameterValue=${aws_secret_access_key} --capabilities CAPABILITY_IAM") }
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