pipeline {
    agent {label 'worker'}
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '4')
        disableConcurrentBuilds()
        timeout(30)
        retry(1)

    }
    parameters {
        booleanParam defaultValue: false, name: 'Run unit tests'
        string defaultValue: 'master', name: 'BRANCH'
        choice choices: ['dev', 'qa', 'prod', 'UAT'], name: 'Environment'
    }
    triggers {
        cron 'H * * * *'
    }
    post {
        always {
            sh "echo always run"
        }
        failure {
            sh "echo failure run"
        }
    }



    stages {
        stage('Git clone') {
            steps{
            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/saiteja199549/example-voting-app.git']])

            }
            
        }
        stage('Build') {
            steps {
                sh '''
                    cd vote
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 458494077457.dkr.ecr.us-east-1.amazonaws.com
                    docker build -t 458494077457.dkr.ecr.us-east-1.amazonaws.com/voteapp:${BUILD_NUMBER} .
                    docker push 458494077457.dkr.ecr.us-east-1.amazonaws.com/voteapp:${BUILD_NUMBER}
                '''
            }
        stage('Deploy') {
            steps {
                sh '''
                    ECR_IMAGE="458494077457.dkr.ecr.us-east-1.amazonaws.com/voteapp:${BUILD_NUMBER}"
                    TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition "vote-app-demo" --region "us-east-1")
                    NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" \'.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)\')
                    NEW_TASK_INFO=$(aws ecs register-task-definition --region "us-east-1" --cli-input-json "$NEW_TASK_DEFINTIION")
                    NEW_REVISION=$(echo $NEW_TASK_INFO | jq \'.taskDefinition.revision\')
                    aws ecs update-service --cluster voteapp --service vote --task-definition --region us-east-1 vote-app-demo:${NEW_REVISION}
'''
            }
        }        
    }
}
}
