node {
  def image
   stage ('checkout') {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/harshalkathar/myfirstapp.git']]])      
        }
   
   stage ('Build') {
         def mvnHome = tool name: 'maven', type: 'maven'
         def mvnCMD = "${mvnHome}/bin/mvn "
         sh "${mvnCMD} clean package"           
        }
    stage('Build image') {
  
       app = docker.build("<gcp-project-id>/springboot")
    }

    stage('Push image to gcr') {
        docker.withRegistry('https://gcr.io', 'gcr:gcp') {
            app.push("${env.BUILD_NUMBER}")
        }
    }

    stage('Update GIT') {
            script {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') 
                {
                    withCredentials([usernamePassword(credentialsId: 'gitlogin', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        def encodedPassword = URLEncoder.encode("$GIT_PASSWORD",'UTF-8')
                        sh "git config user.email katharharshal1@gmail.com"
                        sh "git config user.name harshalkathar"
                        sh "sed -i 's+<gcp-project-id>/springboot.*+<gcp-project-id>/springboot:${env.BUILD_NUMBER}+g' spring-boot.yaml"
                        sh "git add ."
                        sh "git commit -m 'jenkinsbuild: ${env.BUILD_NUMBER}'"
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/myfirstapp.git HEAD:master"
                }
                    
                  }
                
            }
    }        
}
