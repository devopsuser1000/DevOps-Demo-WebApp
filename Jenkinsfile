pipeline {
    agent any
    stages {
        stage('CodeCheckout') {
            steps {
                slackSend channel: 'alerts', message: 'Discovery phase pipeline test'
            }
        }
    }
}