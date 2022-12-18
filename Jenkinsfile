pipeline{
    //agent any
    agent {label 'worker'}
    options{
        buildDiscarder(logRotator(daysToKeepStr: '15'))
        disableConcurrentBuilds()
        timeout(time: 5, unit: 'MINUTES')
        retry (3)
    }
    parameters{
        string(name: 'BRANCH', defaultValue: 'master')
        booleanParam(name: 'Critical', defaultValue: false)
        choice(name: 'Environment', choices: ['Dev', 'Qa', 'Uat', 'Prod'])
    }
    environment{
        ECS_CLUSTER = "vote-app"
        ENVIRONMENT = "global"
    }
    stages{
       stage('Git Checkout'){
        steps{
            checkout scm
        }
       } 
       stage('Build and Push'){
        steps{
            sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 017706783036.dkr.ecr.us-east-1.amazonaws.com"
            sh "cd vote && docker build -t 266948012606.dkr.ecr.us-east-1.amazonaws.com/newcicd:v${BUILD_NUMBER} ."
            sh "docker push 266948012606.dkr.ecr.us-east-1.amazonaws.com/newcicd:v${BUILD_NUMBER}"
        }
       }
       stage('Deploy Stage'){
        steps{
            sh '''
            ECR_IMAGE="266948012606.dkr.ecr.us-east-1.amazonaws.com/newcicd:v${BUILD_NUMBER}"
            TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition vote --region us-east-1)
            NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')
            NEW_TASK_INFO=$(aws ecs register-task-definition --region us-east-1 --cli-input-json "$NEW_TASK_DEFINTIION")
            NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
            aws ecs update-service --region us-east-1 --cluster vote-app --service vote  --task-definition vote:${NEW_REVISION}'''
        }
       } 
    }
    post{
        always{
            sh "echo running final steps"
        }
    }

}
