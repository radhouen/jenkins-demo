#!groovy
environment {
     VERSION = "0.1"
   }
tools {nodejs "node"}
properties(
    [
        [$class: 'BuildDiscarderProperty', strategy:
          [$class: 'LogRotator', artifactDaysToKeepStr: '14', artifactNumToKeepStr: '5', daysToKeepStr: '30', numToKeepStr: '60']],
        pipelineTriggers(
          [
              pollSCM('H/15 * * * *'),
              cron('@daily'),
          ]
        )
    ]
)
node {
    stage('Checkout') {
        deleteDir()
        checkout scm
    }

     stage('Check NPM version') {
        withEnv(["NPM_CONFIG_LOGLEVEL=warn"]) {
            sh 'npm config ls'
        }
    }

    stage('NPM Install') {
        withEnv(["NPM_CONFIG_LOGLEVEL=warn"]) {
            sh 'npm install'
        }
    }

   stage('Build') {
        milestone()
        sh 'npm run prod'
    }

    stage('Archive') {
        sh 'tar -cvzf dist.tar.gz --strip-components=1 dist'
        archive 'dist.tar.gz'
    }

  stage('Backup') {
        milestone()
        echo "Backup last version..."
        echo '/var/www/html/jenkins-demo/$(date +%F_%H.%M.%S)/'
        sh 'min=/var/www/html/jenkins-demo/$(date +%F_%H.%M.%S)/; mkdir $min; cp -arf /var/www/html/jenkins-demo/. $min'
    }
 
 stage('Deploy') {
        milestone()
        echo "Deploying..."
        sh 'min=/var/www/html/jenkins-demo/;cp -arf dist/. $min; cp /var/www/html/robots.txt $min'
    }
}
