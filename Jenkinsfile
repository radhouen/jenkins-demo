#!groovy
environment {
     VERSION = "0.1"
   }
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
    
    env.NODEJS_HOME = "${tool 'NodeJS 10.15.0'}"
    // on linux / mac
    env.PATH="${env.NODEJS_HOME}/bin:${env.PATH}"
    // on windows
    env.PATH="${env.NODEJS_HOME};${env.PATH}"
    sh 'npm --version'

    stage('Checkout') {
        //disable to recycle workspace data to save time/bandwidth
        deleteDir()
        checkout scm

        //enable for commit id in build number
        //env.git_commit_id = sh returnStdout: true, script: 'git rev-parse HEAD'
        //env.git_commit_id_short = env.git_commit_id.take(7)
        //currentBuild.displayName = "#${currentBuild.number}-${env.git_commit_id_short}"
    }

     stage('Check NPM version') {
        steps {
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
