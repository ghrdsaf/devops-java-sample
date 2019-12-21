def label = "mypod-${UUID.randomUUID().toString()}"
def registry = "192.168.56.46:8050"
def app_name = "javatest"
def namespace = "jenkins"
def username = "xxxxxxxxxxxx"
def regpass = "xxxxxxxxx"
def k8s_auth = "23dfee3c-9b8f-4567-b330-aee926724b09"
def gitlab_auth = "5d59d041-41cb-49e7-afef-be6c73b412df"
def gitlab_url = "http://gitlab:3086/liuyi/test1"

podTemplate(label: 'label', cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'jnlp', 
        image: 'myjnlp-slave:1.0'
    ),
],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    hostPathVolume(mountPath: '/usr/bin/docker', hostPath: '/usr/bin/docker')
],)
{
    node('label') {
        stage('Task') {
            stage('拉取代码') {
                git credentialsId: "${gitlab_auth}", url: "${gitlab_url}"
                def mytag = sh returnStdout: true, script: 'git describe --always --tag'
                sh "git checkout -b $mytag"
                echo "mytag $mytag ${mytag} ----"
            }
            stage('编译打包') {
                sh '''
				#sh "mvn clean package -Dmaven.test.skip=true"
				docker ps
				docker images
				'''
            }
            
            stage('构建上传镜像') {
                def mytag = sh returnStdout: true, script: 'git describe --always --tag'
                def image_name = "${app_name}:${mytag}".minus("\n")
                
                echo "image_name $image_name"
                sh label: '', script: '''              echo \'
                    FROM tomcat:latest 
                    ADD target/*.jar /usr/local/tomcat/webapps/ 
                    \' > Dockerfile
                ''' 
                    
                sh """
                    docker build  -t "${registry}/${namespace}/${image_name}" ./
                    docker login -u ${username} -p \"${regpass}\" ${registry}
                    docker push ${registry}/${namespace}/${image_name}
                    
                """
            }
             stage('部署到K8S'){
                def mytag = sh returnStdout: true, script: 'git describe --always --tag'
                def image_name = "${app_name}:${mytag}".minus("\n")
                sh """
                sed -i 's#tomcat:latest#${registry}/${namespace}/${image_name}#' java-deploy.yaml
                """
                kubernetesDeploy configs: 'java-deploy.yaml', kubeconfigId: "${k8s_auth}"
      }
        }
    }
}
