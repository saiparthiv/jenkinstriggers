def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
      maven "MAVEN3"
      jdk "OracleJDK11"
  }

    environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = "805619463928.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://805619463928.dkr.ecr.us-east-1.amazonaws.com"
    }
  stages {
    stage('Fetch Code'){
      steps {
        git branch: 'master', url: 'https://github.com/saiparthiv/User-Control-Panel.git'
      }
    }


    stage('Test'){
      steps {
        sh 'mvn test'
      }
    }

    stage ('Code Analysis with Checkstyle'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

    stage('SonarQube Analysis') {
      steps {
        script {
            def scannerHome = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            withSonarQubeEnv('sonar') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
        }
      }
    }


    stage('Build App Image') {
       steps {
       
         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/")
             }

     }
    
    }

    stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
    }

  }
  post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
