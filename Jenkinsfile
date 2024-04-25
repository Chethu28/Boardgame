pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk    "jdk17"
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCESS_KEY_ID = credentials("access_key")
        AWS_SECRET_ACCESS_KEY = credentials("secret_id")
        AWS_DEFAULT_REGION="us-east-1"
    }
    

    stages {
        stage('git-checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Chethu28/Boardgame.git'
            }
        }
        
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('scan-file-system') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        
        stage('sonarqube-scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('quality-gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
    
        stage('upload-artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'nexus-config', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('Build image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t chethanreddy28/boardgame:${BUILD_NUMBER} .'
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html chethanreddy28/boardgame:${BUILD_NUMBER} "
            }
        }
        
        stage('image-uploader') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh 'docker push chethanreddy28/boardgame:${BUILD_NUMBER}'
                }
            }
        }
        
        stage("Delpoy-to-eks") {
            steps {
                sh "aws eks update-kubeconfig --name boardgame --region us-east-1"
                sh "kubectl apply -f deployment-service.yaml "
                sleep 120                
                sh "kubectl get po"
                sh "kubectl get svc"
            }
        }
        
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'chethanreddy.mp@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-fs-report.html'
            )
        }
    }
}

}
