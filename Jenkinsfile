pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
                name: kaniko
            spec:
                restartPolicy: Never
                volumes:
                - name: kaniko-secret
                  secret:
                    secretName: kaniko-secret
                containers:
                - name: kaniko
                  image: gcr.io/kaniko-project/executor:debug
                  command:
                    - /busybox/cat
                  tty: true
                  volumeMounts:
                  - name: kaniko-secret
                    mountPath: /kaniko/.docker
            """
        }
    }

    stages {
        stage('Cloning the repo') {
            steps {
                script {
                    // Clone the repository
                    git branch: 'main', url: 'https://github.com/amanravi-squareops/zipkin-server'
                }
            }
        }
stage('kaniko build & push') {
    steps {
        container('kaniko') {
            script {
                def a = sh(
                    script: 'date "+%b-%d-time-%H:%M" | cut -c 1-16',
                    returnStdout: true
                ).trim()
                
                def imageTag = "${a}-${BUILD_NUMBER}"
                
                sh '''
                /kaniko/executor --dockerfile /Dockerfile \
                --context=$(pwd) \
                --destination=amanravi12/zipkin-server:${imageTag}
                '''
            }
        }
    }
}

        stage('Update values.yaml') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-cre', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        git branch: 'main', 
                            url: "https://${USERNAME}:${PASSWORD}@github.com/amanravi-squareops/springboot-helm.git"
                    }
                    sh '''
                    cd zipkin-server
                    sed -i "s/tag: .*/tag: ${BUILD_NUMBER}/" values.yaml
                    cat values.yaml
                    git config --global user.email "aman.ravi@squareops.com"
                    git config --global user.name "amanravi-squareops"
                    git add values.yaml
                    git commit -m "Update imageTag in values.yaml"
                    git push origin main
                    '''
                }
            }
        }
    }
}
