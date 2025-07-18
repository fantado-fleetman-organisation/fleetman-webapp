pipeline {
   agent any
   parameters {
    string(name: 'SERVICE_NAME', defaultValue: 'fleetman-webapp', description: 'Microservice name')
    string(name: 'ECR_REPO_NAME', defaultValue: 'fleetman-webapp', description: 'ECR repo name')
    string(name: 'AWS_REGION', defaultValue: 'us-east-1', description: 'AWS Region for ECR')
   }
   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     //SERVICE_NAME = "fleetman-webapp"
     //REPOSITORY_TAG="${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
     AWS_ACCOUNT_ID = credentials('aws-account-id') // Use Jenkins credential binding (string)
     AWS_ACCESS_KEY_ID = credentials('aws-access-key')
     AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
     IMAGE_TAG = "${AWS_ACCOUNT_ID}.dkr.ecr.${params.AWS_REGION}.amazonaws.com/${params.ECR_REPO_NAME}:${BUILD_ID}"
  }
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/fantado-fleetman-organisation/${params.SERVICE_NAME}.git"
         }
      }
      stage('Build') {
         steps {
            sh 'echo No build required for Webapp.'
         }
      }
      stage('Login to ECR') {
      steps {
        sh """#!/bin/bash
          aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
          aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
          aws configure set region ${params.AWS_REGION}
          aws ecr get-login-password --region ${params.AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${params.AWS_REGION}.amazonaws.com
        """
          }
        }
      stage('Build and Push Image') {
         steps {
           //sh 'docker image build -t ${REPOSITORY_TAG} .'
            sh '''
          docker build -t $IMAGE_TAG .
          docker push $IMAGE_TAG
            '''
         }
      }

      stage('Deploy to Cluster') {
          steps {
            //sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
            sh """#!/bin/bash
            export REPOSITORY_TAG=$IMAGE_TAG
            envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -
           """
          }
      }
   }
}

