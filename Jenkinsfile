pipeline {
 agent {
   label 'maven'
  }
    parameters { 
        string(name: 'BUILD_NUMBER',defaultValue: '1.0',description: '')
        string(name: 'APP_NAME',defaultValue: 'devops-sample',description: '')
    }

    environment {
        DOCKER_CREDENTIAL_ID = 'aliyun-id'
        GITEE_CREDENTIAL_ID = 'gitee-id'
        KUBECONFIG_CREDENTIAL_ID = 'demo-kubeconfig'
        REGISTRY = 'registry.cn-hangzhou.aliyuncs.com'
        DOCKERHUB_NAMESPACE = 'liuyik8s'
        GITEE_ACCOUNT = 'liuyik8s'
        BRANCH_NAME = 'master'
        }

    stages {
        stage ('checkout scm') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'gitee-id', url: 'https://gitee.com/liuyik8s/devops-java-sample.git']]])
  
            }
        }
 
        stage('checking'){
            steps {
                sh '''
                pwd
                echo "webhook"
                echo "webhook"
                ls -l
                sleep 2
                echo "$[app]"
                '''
            }    
      }
        stage('Unit Testing'){
          steps {
            echo "Unit Testing..."
          }
    }
    
       stage ('unit test') {
            steps {
                container ('maven') {
     //              sh 'mvn clean  -gs `pwd`/configuration/settings.xml test'
                }
            }
        }
 
        stage ('build & push') {
            steps {
                container ('maven') {
                    sh 'mvn  -Dmaven.test.skip=true -gs `pwd`/configuration/settings.xml clean package'
                    sh 'sleep 1'
                    sh 'env'
                    sh 'ls -l target'
                    sh 'docker build -f Dockerfile-online -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
                    withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
                    }
                }
            }
     }
    
        stage('push latest'){
           when{
             branch 'master'
           }
           steps{
                container ('maven') {
                  sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                  sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                }
           }
        }
    
       stage('deploy to dev') {
          when{
            branch 'master'
          }
          steps {
       //     input(id: 'deploy-to-dev', message: 'deploy to dev?')
       //     kubernetesDeploy(configs: 'deploy/dev-ol/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
              sh "ls -l deploy/dev-ol"
              sh "kubectl apply -f deploy/dev-ol/*"
          }
        }

        stage('push with tag'){
          when{
            expression{
              return params.TAG_NAME =~ /v.*/
            }
           }
          steps {
              container ('maven') {
         //       input(id: 'release-image-with-tag', message: 'release image with tag?')
                  withCredentials([usernamePassword(credentialsId: "$GITHUB_CREDENTIAL_ID", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh 'git config --global user.email "kubesphere@yunify.com" '
                    sh 'git config --global user.name "kubesphere" '
                    sh 'git tag -a $TAG_NAME -m "$TAG_NAME" '
                    sh 'git push http://$GIT_USERNAME:$GIT_PASSWORD@github.com/$GITHUB_ACCOUNT/devops-java-sample.git --tags --ipv4'
                  }
                sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
                sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
          }
          }
        }
 
       stage('deploy to production') {
          when{
            expression{
              return params.TAG_NAME =~ /v.*/
            }
          }
          steps {
           // input(id: 'deploy-to-production', message: 'deploy to production?')
            kubernetesDeploy(configs: 'deploy/prod-ol/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
          }
        }
}
}
