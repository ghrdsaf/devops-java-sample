pipeline {

  agent {
   label 'maven'
  }
    
   parameters { 
        string(name: 'TAG_NAME',defaultValue: '1.3',description: '')
        choice choices: ['devops-java-sample', 'gateway', 'order', 'product'], name: 'APP_NAME'
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
                    sh 'docker build -f Dockerfile-online -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$TAG_NAME .'
                    withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$TAG_NAME'
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
                  sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$TAG_NAME $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                  sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                }
           }
       }
      
       stage('deploy to dev') {
          when{
            branch 'master'
          }
          steps {
             container ('maven'){
       //     input(id: 'deploy-to-dev', message: 'deploy to dev?')
              sh "ls -l deploy/dev-ol"
              sh "sleep 1"
              sh "ls -l deploy/dev-ol"
              sh "cat deploy/dev-ol/*.yaml"
              sh "sleep 1"
             
             sh '''
             echo "changing parameter"
             cp deploy/dev-ol/devops-sample.yaml k8s.yaml
             
             sed -i "s#TAG_NAME#$BUILD_NUMBER#g" k8s.yaml
             sed -i "s#REGISTRY#$REGISTRY#g" k8s.yaml
             sed -i "s#DOCKERHUB_NAMESPACE#$DOCKERHUB_NAMESPACE#g" k8s.yaml
             sed -i "s#APP_NAME#$APP_NAME#g" k8s.yaml
             sed -i "s#BRANCH_NAME#$BRANCH_NAME#g" k8s.yaml

             '''
           //  sh 'kubectl delete -f k8s.yaml -n testing'
             sh "sleep 10"
             sh "cat k8s.yaml"
             sh 'kubectl apply -f k8s.yaml -n testing'
           }
          }
        }

        stage('push with tag'){
          
            steps {
              container ('maven') {
         //       input(id: 'release-image-with-tag', message: 'release image with tag?')
                  withCredentials([usernamePassword(credentialsId: "$GITEE_CREDENTIAL_ID", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh 'git config --global user.email "liuyi71@sina.com" '
                    sh 'git config --global user.name "liuyi71k8s" '
                    sh 'git tag -a $TAG_NAME -m "$TAG_NAME" '
                    sh 'git push http://$GIT_USERNAME:$GIT_PASSWORD@gitee.com/$GITEE_ACCOUNT/$APP_NAME.git --tags --ipv4'
                    }
            
                  sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$TAG_NAME $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
                  sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
               }
           }
        }
 
       stage('deploy to production') {
       
          steps {
           container ('maven') {
            input(id: 'deploy-to-production', message: 'deploy to production?')
           // sh 'kubectl delete -f k8s.yaml -n production'
             sh "env"
             sh "sleep 10"
             sh "cat k8s.yaml"
             sh 'kubectl apply -f k8s.yaml -n production'
          }
        }
     }

    }


}
