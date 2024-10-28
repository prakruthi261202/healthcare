pipeline {
    agent any
    tools {
        maven 'M2_HOME'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                echo 'This stage is to clone the repo from github'
                git branch: 'master', url: 'https://github.com/prakruthi261202/healthcare.git/'
            }
        }
        stage('Create Package') {
            steps {
                echo 'This stage will compile, test, package my application'
                sh 'mvn package'
            }
        }
        stage('Generate Test Report') {
            steps {
                echo 'This stage generates Test report using TestNG'
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false,
                    reportDir: '/var/lib/jenkins/workspace/Healthcare/target/surefire-reports',
                    reportFiles: 'index.html',
                    reportName: 'HTML Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                ])
            }
        }
        stage('Create Docker Image') {
            steps {
                echo 'This stage will create a Docker image'
                sh 'docker build -t prakruthi810/healthcare:1.0 .'
            }
        }
    }
}
