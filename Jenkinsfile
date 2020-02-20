pipeline {

    agent none

    stages {
        stage('Build Masterimage') {
            agent { label 'docker'
            steps {
                cd qmstr-master
                make masterimage
            }
        }
    }
}
