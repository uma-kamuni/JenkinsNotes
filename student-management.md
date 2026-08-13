1.Setup Jenkins Master-Agent
2.Create jenkins job
job name: application-pipeline

vim jenkinsfile
'''
pipeline {
    agent {
        label "worker"
    }

    environment {
        my_aws_credentials = credentials('aws-access')
    }

    tools {
        maven "maven"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/mayurpandit25/student-management.git'
                sh '''
                echo "Cloning is successful"
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                mvn clean package
                echo "Build is successful"
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
                }
            }
        }

        stage('Artifact to S3') {
            steps {
                sh '''
                aws s3 cp target/*.war s3://umak.online/Artifact/student.war
                '''
            }
        }
    }

    post {
        success {
            sh '''
            echo 'Pipeline is successful'
            '''
        }

        failure {
            echo 'Pipeline is failed'
        }
    }
}
'''
#Before building thsi job 
1.Install Maven plugin
2.Configure Tools in jenkins maanage --> Tools Section
3.Install aws cli in worker node 
'''
sudo snap install aws-cli --classic
'''

4. install AWS Credential plugin 
'''
Jenkins UI-->manage jenkins --> Credentials---> Global---> Aws credentials--->
name : aws-access
accessKey:
SecretAccessKey:
save
'''

5.Create S3 bucket in aws s3 to store artifacts
'''
name : umak.online
make bucket public
versioning: enable
'''

6.In worker node install java jdk-21 and switch to java jdk-21

7. build jenkins job
--> it will store artifact in s3
