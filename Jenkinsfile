pipeline {
    agent any
    tools {
        maven 'M2_HOME'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'This stage is to clone the repo from GitHub'
                git branch: 'master', url: 'https://github.com/prakruthi261202/healthcare.git/'
            }
        }
        stage('Create Package') {
            steps {
                echo 'This stage will compile, test, and package my application'
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
        stage('Login to Dockerhub') {
            steps {
                echo 'This stage will log into Dockerhub'
                withCredentials([usernamePassword(credentialsId: 'dockerloginnew', passwordVariable: 'dockerpass', usernameVariable: 'dockeruser')]) {
                    sh 'docker login -u ${dockeruser} -p ${dockerpass}'
                }
            }
        }
        stage('Docker Push-Image') {
            steps {
                echo 'This stage will push my new image to Dockerhub'
                sh 'docker push prakruthi810/healthcare:1.0'
            }
        }
        stage('AWS-Login') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Awsaccess', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    echo 'Logged into AWS'
                }
            }
        }
        stage('Terraform Operations for Test Workspace') {
            steps {
                script {
                    sh '''
                        terraform workspace select test || terraform workspace new test
                        terraform init
                        terraform plan
                        terraform destroy -auto-approve
                    '''
                }
            }
        }
        stage('Terraform Destroy & Apply for Test Workspace') {
            steps {
                sh 'terraform apply -auto-approve'
            }
        }
        stage('Get Kubeconfig') {
            steps {
                sh '''
                source /home/ubuntu/myenv/bin/activate
                /home/ubuntu/myenv/bin/aws eks update-kubeconfig --region us-east-1 --name test-cluster
                /home/ubuntu/myenv/bin/aws eks get-token --cluster-name test-cluster
                '''
                sh 'kubectl get nodes'
            }
        }
        stage('Deploying the Application') {
            steps {
                sh 'kubectl apply -f app-deploy.yml'
                sh 'kubectl get svc'
            }
        }
        stage('Terraform Operations for Production Workspace') {
            when {
                expression {
                    return currentBuild.currentResult == 'SUCCESS'
                }
            }
            steps {
                script {
                    sh '''
                        terraform workspace select prod || terraform workspace new prod
                        terraform init
                        terraform plan
                        terraform destroy -auto-approve
                    '''
                }
            }
        }
        stage('Terraform Destroy & Apply for Production Workspace') {
            steps {
                sh 'terraform apply -auto-approve'
            }
        }
        stage('Get Kubeconfig for Production') {
            steps {
                sh '''
                source /home/ubuntu/myenv/bin/activate
                /home/ubuntu/myenv/bin/aws eks update-kubeconfig --region us-east-2 --name prod-cluster
                /home/ubuntu/myenv/bin/aws eks get-token --cluster-name prod-cluster
                '''
                sh 'kubectl get nodes'
            }
        }
        stage('Deploying the Application to Production') {
            steps {
                sh 'kubectl apply -f app-deploy.yml'
                sh 'kubectl get svc'
            }
        }
    }
}
