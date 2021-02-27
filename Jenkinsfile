pipeline {
    agent any
    stages {
        stage('CodeCheckout') {
            steps {
                slackSend channel: 'alerts', message: 'Discovery phase pipeline test'
                slackSend channel: 'alerts', message: 'Project checkout from Git'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/devopsuser1000/DevOps-Demo-WebApp.git']]])
                slackSend channel: 'alerts', message: 'Git checkout complete'
            }
        }
        stage('BuildProject') {
            steps {
                slackSend channel: 'alerts', message: 'Building project...'
                sh 'mvn clean install'
            }
        }
        stage('DeployToTest') {
            steps {
                slackSend channel: 'alerts', message: 'Deploy the Application to the Test environment'
                sshagent(['deploy_user']) {
                    sh "scp -o StrictHostKeyChecking=no target/AVNCommunication-1.0.war ubuntu@ec2-3-17-207-180:/var/lib/tomcat8/webapps/QAWebapp.war"
                    sh "scp -o StrictHostKeyChecking=no -r target/AVNCommunication-1.0 ubuntu@ec2-3-17-207-180:/var/lib/tomcat8/webapps/QAWebapp"
            }
        }
       }
       stage('UI Test') {
            steps {
                slackSend channel: 'alerts', message: 'Starting UI Tests...'
                sh 'mvn -f functionaltest/pom.xml test'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
            }
       }
    }
}