pipeline {
  environment {
    GIT_COMMIT_SHORT = "${env.GIT_COMMIT.take(7)}"
    BRANCH_NAME_LOWER = "${BRANCH_NAME.toLowerCase()}"
    IMAGE_REPO = '765119597117.dkr.ecr.eu-west-1.amazonaws.com'
    APP_NAME = "test-app"
    IMAGE_TAG = "${IMAGE_REPO}/${APP_NAME}:${env.BRANCH_NAME_LOWER}"
  }
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          securityContext:
            fsGroup: 1000
            runAsUser: 0
          serviceAccountName: jenkins
          nodeSelector:
            service: jenkins-agent
          containers:
          - name: kaniko
            image: gcr.io/kaniko-project/executor:v1.6.0-debug
            imagePullPolicy: Always
            command:
            - /busybox/cat
            tty: true
            resources:
              requests:
                memory: 2Gi
                cpu: 1
              limits:
                memory: 4Gi
                cpu: 4
          - name: nodejs
            image: 376641251557.dkr.ecr.eu-west-1.amazonaws.com/node:14.7.0-slim
            command:
              - cat
            tty: true
            resources:
              requests:
                memory: 4Gi
                cpu: 1
              limits:
                cpu: 4
          - name: kubectl
            image: 376641251557.dkr.ecr.eu-west-1.amazonaws.com/jenkins-worker:4.1.2
            command:
              - cat
            tty: true
            resources:
              requests:
                memory: 0.25Gi
                cpu: 0.25
              limits:
                memory: 0.25Gi
                cpu: 0.25
        '''
      }
  }
     stages {
         stage('Build') {
             steps {
                 sh 'echo "Hello World"'
                 sh '''
                     echo "Multiline shell steps works too"
                     ls -lah
                 '''
             }
         }      
         stage('Download from aws') {
              steps {
                  withAWS(region:'eu-west-1',credentials:'itservice-test-id') {
				  			    container('kubectl'){
                  sh 'echo "Downloading content with AWS creds"'
                      s3Download(file:'visual match.zip', bucket:'amytest123', path:'visual match.zip', force:true)
					  sh 'ls -ltrh'  
                  }
				  }
              }
         }
         stage('File check') {
              steps {
                  withAWS(region:'eu-west-1',credentials:'itservice-test-id') {
				    container ('kaniko') {
               sh label: 'Auth to ECR', script: '/busybox/echo "{\\"credHelpers\\":{\\"$IMAGE_REPO\\":\\"ecr-login\\"}}" > /kaniko/.docker/config.json'
                sh label: 'Build image with kaniko', script: '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=${IMAGE_TAG} --destination=${IMAGE_TAG}-${GIT_COMMIT_SHORT}'
					  sh 'ls -ltrh' 
					  
                  }
				  }
              }
         }		 
     }
}
