node {
  def image
   stage ('checkout') {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/Sumanth17-git/myfirstapp.git']]])      
        }
   
   stage ('Build') {
         def mvnHome = tool name: 'maven', type: 'maven'
         def mvnCMD = "${mvnHome}/bin/mvn "
         sh "${mvnCMD} clean package"           
        }
    stage('Build image') {
  
       app = docker.build("decent-creek-369909/springboot")
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
			sh "git remote set-url origin git@github.com:Sumanth17-git/myfirstapp.git"
			sh "git remote add origin git@github.com:Sumanth17-git/myfirstapp.git"
                        sh "git config user.email sumanth.suman17@gmail.com"
                        sh "git config user.name sumanth17"
                        sh "sed -i 's+decent-creek-369909/springboot.*+decent-creek-369909/springboot:${env.BUILD_NUMBER}+g' spring-boot.yaml"
                        sh "git add -A"
                        sh "git commit -m 'jenkinsbuild: ${env.BUILD_NUMBER}'"
                        sh "git push origin main"
                }
                    
                  }
                
            }
    }        
}
