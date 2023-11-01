pipeline {
    agent any

    stages{
    stage('Git Checkout') {
        steps{
        git branch:'feature-rizme' , changelog: false, credentialsId: 'riz_jenken', poll: false, url: 'https://github.com/IBT-learning/ibt-maven.git'
        }
    }
    stage('Validate') {
        steps{
            withMaven(maven: 'maven_3.8') {
               sh 'mvn validate'
            }
        }
    }
    stage('Compile'){
        steps{
           withMaven(maven: 'maven_3.8') {
                sh 'mvn compile'
            }
        }
    }
     stage('Test'){
        steps{
           withMaven(maven: 'maven_3.8') {
                sh 'mvn test'
            }
        }
    }
    stage('SonarQube Analysis'){
        environment{
            sonarScan = tool 'ibt-sonarqube_4.8'
        }
        steps{
            withSonarQubeEnv(credentialsId: 'student-sonar-token', installationName: 'IBT sonarqube') {
               sh "${env.sonarScan}/bin/sonar-scanner"
            }
        }
    }
    stage('Package'){
            steps{
               withMaven(maven: 'maven_3.8') {
                    sh 'mvn package'
                }
            }
    }
    stage('Dynamic Scan') {
        steps{
            dependencyCheck additionalArguments: '''
                    -o "./"
                    -s "./"
                    -f "ALL"
                    --prettyPrint ''', odcInstallation: 'dependency-check'
            dependencyCheckPublisher pattern: 'dependency-check-report.xml'
        }
    }
    stage('Upload to Artifactory'){
        steps{
            withMaven(maven: 'maven_3.8') {
                  withCredentials([file(credentialsId: 'mvn_settings_riz', variable: 'mvn_settings_rizme')]) {

                    sh 'mvn deploy -s $mvn_settings_rizme'
                }
            }
        }
    }
    stage('Upload to Artifactory (configFile)'){
        steps{
            withMaven(maven: 'maven_3.8') {
                configFileProvider([configFile(fileId: 'artifactory-settings', targetLocation: 'mvn_settings_managed', variable: 'mvn_settings_managed')]) {
                        sh 'mvn deploy -s $mvn_settings_managed'
                }
            }
        }
    }
    stage('Install Tomcat using playbook'){
        steps{
            ansiblePlaybook(
                        become: true,
                        credentialsId: 'server-ssh-pwd',
                        inventory: 'inventory_sept',
                        playbook: 'tomcat_sept.yaml'
                                              )
        }
    }
    stage('Deploy to Dev'){
    //when {
      //      expression{
        //        env.BRANCH_NAME=='main'
          //  }
    // }
        steps{
            script{
                def remote = [name: 'Dev' , host: '167.99.62.93' , allowAnyHosts: true]
                withCredentials(usernamePassword(credentialsId: 'server-ssh-pw', passwordVariable: 'password', usernameVariable: 'username')) {
                    remote.user = username
                    remote.password = password
                  sshPut remote: remote, from: 'target/ibt-maven-2.22-SNAPSHOT.jar', into: 'opt/tomcat/apps'
                }
            }
        }
    }
 }
}