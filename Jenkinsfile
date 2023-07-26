pipeline {

    options {
      buildDiscarder(logRotator(numToKeepStr: '30', daysToKeepStr:'10'))
    }
    
    // tools{
    //     jdk "java8"
    //     maven "mvn-3.3.9"
    // }

    agent { node { label 'master' } }
    
    environment {
        //SonarQube Configs
        projectKey = "example-helloworld"
        serverUrl = "https://etsonar.axiatadigitallabs.com"
        projectToken = "7d9aee4f1478be7d2fbfb639374db51a9ad5ffa1"

        //GitLab Repo
       //gitlabUrl ="https://gitlab.axiatadigitallabs.com/axp/egw-ussd-ms"

        //GitLab Credentials ID
        //credentialsId= "CICI_AXP"
    }

    stages {
        stage('Clone') { 
            steps {
                sh 'mvn -v'
                sh 'java -version'
                sh 'git clone https://github.com/kuMunasinghe/mvn-helloworld.git'
            }
        }
        stage('Build'){
            steps{
                dir('/var/lib/jenkins/workspace/mvn-helloworld/mvn-helloworld'){
                    sh 'mvn clean install' 
                }

            }
        }
        stage('Unit test') {
            steps {
                dir('/var/lib/jenkins/workspace/mvn-helloworld/mvn-helloworld'){
                    sh 'mvn test'
                }
            }
        }
        stage('JUnit'){
            steps{
                dir('/var/lib/jenkins/workspace/mvn-helloworld/mvn-helloworld')
                {
                    junit '**/target/surefire-reports/*.xml'
                } 
            }
        }
        stage('SonarQube Analysis'){
            options {
                timeout(time: 10, unit: 'MINUTES')   // timeout on whole pipeline job
                }         
            steps{
             script{
                try{
                                    dir('/var/lib/jenkins/workspace/mvn-helloworld/mvn-helloworld'){
                     sh "mvn sonar:sonar -Dsonar.projectKey=${env.projectKey} -Dsonar.host.url=${env.serverUrl} -Dsonar.login=${env.projectToken}"
                                    }
                }
                catch (Exception e) {
                    echo 'Exception occurred: ' + e.toString()
                    echo 'echo SonarQube Analysis Failed !'
                }
                }
                
                
            }
        }
        stage('Push to Registry'){
            steps {
                script{
                    try{
                        echo 'Deploying only because this commit is build from master/tagged...'
                        configFileProvider([configFile(fileId: 'wso2-nexus-global', variable: 'MAVEN_SETTINGS')]) {
                dir('/var/lib/jenkins/workspace/mvn-helloworld/mvn-helloworld'){
                                            sh 'mvn -s $MAVEN_SETTINGS clean deploy -U'    
                }

                        }
                    }
                    catch (Exception e) {
                            echo 'Exception occurred: ' + e.toString()
                            echo 'echo Maven Deploy Failed !'
                }
            }
            }
        }
  }
   post {
	    success{
                script {
                    def causes = currentBuild.rawBuild.getCauses()
                    for(cause in causes) {
                        if (cause.class.toString().contains("UpstreamCause")) {
                            println "This job was caused by job " + cause.upstreamProject
                        } else {
                            println "Root cause : " + cause.toString()
    //                        build job: '02-ADL-ET-AXP-SRE-COMPONENT-DEP', wait: false
                        }
                    }
                }
    //         mail to: "${gitlabUserEmail}",
    //         subject: "Core Util: ${currentBuild.currentResult}: ${env.JOB_NAME}",
    //         body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\n\n More Info can get by : ${env.BUILD_URL}\n\n "
         }
	   // failure{
    //         mail to: "${gitlabUserEmail}",
    //         subject: "Core Util: ${currentBuild.currentResult}: ${env.JOB_NAME}",
    //         body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\n\n More Info can get by : ${env.BUILD_URL}\n\n "
    //     }        
         always {
             cleanWs()
        }
   }
}

