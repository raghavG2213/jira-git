properties([pipelineTriggers([githubPush()])])

pipeline {
     agent any
  tools {
    maven 'Maven'
  }
  environment {
   registry = "rgdocker2213/spring_test"
   registryCredential = "4a20d5b1-f901-4d7e-a5f4-19ea550194fc"
  }
       
      
  stages {
    stage('Pipeline Trigger') {
            steps {
                checkout([
                 $class: 'GitSCM',
                 branches: [[name: 'master']],
                 userRemoteConfigs: [[
                    url: 'https://github.com/raghavG2213/jira-git.git',
                    credentialsId: '',
                 ]]
                ])
            }
     }   
       
    stage('Initialize'){
      steps{
        echo "We are doing some tests"
        echo "PATH = ${PATH}"
        }
    }
    stage('Build'){
           steps
           {
        sh "mvn clean install"
    }     
    }
      
stage('Image Build') { 
             steps { 
                    script {
                 dockerImage = docker.build registry + ":$BUILD_NUMBER"
                    }
                 } 

     }
      stage('Docker Image Deploy') { 
          steps { 
                 script { 
                  docker.withRegistry( '', registryCredential ) { 
                  dockerImage.push() 
                        }
                   } 
          }
      }
      stage('Application Deploy on K8s') {
             
             steps {
     
                sshagent(['raghavg_1626093']) {
                      sh "scp -o StrictHostKeyChecking=no Spring.yml raghavg_1626093@13.93.120.161:/home/raghavg_1626093/"
                   script
                          {
                                 try
                                 {
                              sh "ssh raghavg_1626093@13.93.120.161 kubectl apply -f Spring.yml -n docker-1626093"   
                                 }
                                 catch(error)
                                 {
                                 sh "ssh raghavg_1626093@13.93.120.161 kubectl create -f Spring.yml -n docker-1626093"   
                                 }
                   }
                     
                }
             }    
     }   
  }
       
       post {
   success {
            echo 'Hello! This was a success!'
              newjira_success()
       }
	failure {
            echo 'Hello! There was a failure!'
              newjira_failure()
       }
    
}
       
  
}

def newjira_success() {
    node {
  stage('JIRA') {
    def testIssue = [fields: [ project: [key: 'JIR'],
                               summary: 'New JIRA Build Success from Jenkins.',
                               description: 'New JIRA Build Success from Jenkins.',
                               issuetype: [name: 'Story']]]

    response = jiraNewIssue issue: testIssue, site: 'jira'

    echo response.successful.toString()
    echo response.data.toString()
  }
}
}

def newjira_failure() {
    node {
  stage('JIRA') {
    def testIssue = [fields: [ project: [key: 'JIR'],
                               summary: 'New JIRA Build Failure from Jenkins.',
                               description: 'New JIRA Build Failure from Jenkins.',
                               issuetype: [name: 'Task']]]

    response = jiraNewIssue issue: testIssue, site: 'jira'

    echo response.successful.toString()
    echo response.data.toString()
  }
}
}



