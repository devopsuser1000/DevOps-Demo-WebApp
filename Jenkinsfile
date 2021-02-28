pipeline {
    agent any
    
 	tools {
        maven 'Maven3.6.3'
   	 }
    
    stages {
        stage('CodeCheckout') {
            steps {
                slackSend channel: 'alerts', message: 'Discovery phase pipeline test'
                slackSend channel: 'alerts', message: 'Project checkout from Git'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/devopsuser1000/DevOps-Demo-WebApp.git']]])
                slackSend channel: 'alerts', message: 'Git checkout complete'
            }
        }
        
        stage('SonarqubeScanner') {
            environment {
                scannerHome = tool 'sonarqube'
            }
            steps {
                slackSend channel: 'alerts', message: 'Static code analysis is in progress'
                withSonarQubeEnv('sonarqube') {
                    sh """${scannerHome}/bin/sonar-scanner"""
                }
               // timeout(time: 2, unit: 'MINUTES') {
               //     waitForQualityGate abortPipeline: true
                //}
                slackSend channel: 'alerts', message: 'Static code analysis is complete'
                }
        }
        
        stage('BuildProject') {
            steps {
                slackSend channel: 'alerts', message: 'Building project...'
       
                //sh 'mvn compile'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }
        stage('DeployToNewTest') {
            steps {
                slackSend channel: 'alerts', message: 'Deploy the Application to the Test environment'
               // sshagent(['deploy_user']) {
               //     sh "scp -o StrictHostKeyChecking=no target/AVNCommunication-1.0.war ubuntu@ec2-18.218.198.155:/var/lib/tomcat8/webapps/QAWebapp.war"
               //     sh "scp -o StrictHostKeyChecking=no -r target/AVNCommunication-1.0 ubuntu@ec2-18.218.198.155:/var/lib/tomcat8/webapps/QAWebapp"
               // }               
               deploy adapters: [tomcat8(url: 'http://35.226.4.247:8080/', credentialsId: 'tomcat', path: '' )], contextPath: '/QAWebapp', war: '**/*.war'
            
        }
       }
       
       
       stage('Artifactory') {
            steps {
                slackSend channel: 'alerts', message: 'Archiving the artifacts..'
                script{
                    // Get the Artifactory server instance details. Defined in the Artifactory Plugin administration page.
                    def server = Artifactory.server "artifactory"
                    // Artifactory Maven instance
                    def rtMaven = Artifactory.newMavenBuild()
                    def buildInfo = Artifactory.newBuildInfo()
                
                        // Tool name from Jenkins configuration
                        rtMaven.tool = "maven"
                        // Artifactory repositories for dependencies resolution and artifacts deployment
                        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
                        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
                        rtMaven.deployer.deployArtifacts = false
                    
                
                        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -e -Dmaven.repo.local=.m2', buildInfo: buildInfo
                
                        server.publishBuildInfo buildInfo
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