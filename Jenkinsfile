pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
         stage('Howdy') {
            steps {
                echo 'howdy'
            }
        } //stage 2
      stage('Download from Git'){
      steps{
           git branch: 'main', changelog: false, credentialsId: 'GitHub_cred_kayshow04', poll: false, url: 'https://github.com/IBT-learning/DevOps-Git.git'
        }
    }
}
