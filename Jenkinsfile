pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java21'
        maven 'Maven3'
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'Github', url: 'https://github.com/kunalrajshah/Jenkins.git'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage('Publish POM') {
          steps {
              sh 'cp pom.xml pom.html'
          }
       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
        
      post {
          success {
              publishHTML([
                 reportDir: '.',
                 reportFiles: 'pom.html',
                 reportName: 'POM Viewer'
             ])
          }
     }
   }
}
