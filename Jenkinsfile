pipeline {
  agent any
 
  stages {
    stage('Install sam-cli') {
      steps {
        bat 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'
        stash includes: '**/venv/**/*', name: 'venv'
      }
    }
    stage('Build') {
      steps {
        unstash 'venv'
        bat 'venv/bin/sam build'
        stash includes: '**/.aws-sam/**/*', name: 'aws-sam'
      }
    }
    stage('beta') {
      environment {
        STACK_NAME = 'sam-app-beta-stage'
        S3_BUCKET = 'sam-jenkins-demo-us-west-2'
      }
      steps {
        withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'us-west-2') {
          unstash 'venv'
          unstash 'aws-sam'
          bat 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
          dir ('hello-world') {
            bat 'npm ci'
            bat 'npm run integ-test'
          }
        }
      }
    }
    stage('prod') {
      environment {
        STACK_NAME = 'sam-app-prod-stage'
        S3_BUCKET = 'sam-jenkins-demo-us-west-2-2'
      }
      steps {
        withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'us-west-2') {
          unstash 'venv'
          unstash 'aws-sam'
          bat 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
        }
      }
    }
  }
}