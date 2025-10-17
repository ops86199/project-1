pipeline {
  agent any

  environment {
    AWS_DEFAULT_REGION = ('us-east-1')          // e.g., ap-south-1 stored as Secret Text
    AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')         // Jenkins credential: Secret Text
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key') // Jenkins credential: Secret Text
    S3_BUCKET = 'omprakshs3bucket0.1'                            // change me
    CF_DISTRIBUTION_ID = 'YOUR_CLOUDFRONT_DISTRIBUTION_ID'       // change me
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Validate Site') {
      steps {
        sh 'ls -la website'
        sh 'test -f website/index.html'
        sh 'test -f website/404.html'
      }
    }

    stage('Deploy to S3') {
      steps {
        dir('website') {
          sh 'aws s3 sync . s3://$S3_BUCKET --delete'
        }
      }
    }

    stage('Invalidate CloudFront Cache') {
      steps {
        sh 'aws cloudfront create-invalidation --distribution-id $CF_DISTRIBUTION_ID --paths "/*"'
      }
    }
  }
}
