pipeline {
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
                      s3Download(file:'test.txt', bucket:'amytest123', path:'test.txt', force:true)
					  sh 'ls -ltrh'
					sh 'cat test.txt'			    
                  }
				  }
              }
         }
         stage('File check') {
              steps {
                  withAWS(region:'eu-west-1',credentials:'itservice-test-id') {
				    container ('kaniko') {
                  sh 'echo "Test"'
                    //  s3Download(file:'test.txt', bucket:'amytest123', path:'test.txt', force:true)
					  sh 'ls -ltrh'  
					    sh 'cat test.txt'
                  }
				  }
              }
         }		 
     }
}
