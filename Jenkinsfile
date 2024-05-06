pipeline {
    agent {
        kubernetes {
            yaml '''
               apiVersion: v1
               kind: Pod
               spec:
                 containers:
                 - name: kaniko
                   image: gcr.io/kaniko-project/executor:debug
                   tty: true
                   command:
                   - sleep
                   args:
                   - infinity
                   volumeMounts:
                     - name: aws-secret
                       mountPath: /root/.aws
                     - name: kube-config
                       mountPath: /root/.kube/config
                 restartPolicy: Never
                 volumes:
                   - name: aws-secret
                     secret:
                       secretName: aws-secret  
                   - name: kube-config
                     configMap:
                       name: kube-config
            '''
        }
    }
    environment {
        ECR_REPO = credentials('deploy-repo')
    }
    stages {
        stage('Clone repository') {
            steps {
                script {
                    checkout scm
                }
            }
        }
        stage('Build and push') {
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    echo "in kaniko" 
                    /kaniko/executor --context `pwd` \
                    --dockerfile Dockerfile \
                    --verbosity debug \
                    --destination=${ECR_REPO}:${BUILD_NUMBER} \
                    --destination=${ECR_REPO}:latest
                    '''
                }
            }
        }
    }
}
