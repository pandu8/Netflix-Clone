pipeline {
    agent any
    
    environment {
        SCANNER_HOME=tool 'sonar-server'
    }
    stages {
        stage('clean workspace') {
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
               git branch: 'main', url: 'https://github.com/pandu8/Netflix-Clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage ("Build Image & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker build -t netflix ."
                        sh "docker tag netflix kalyanvasu08/netflix:latest"
                        sh "docker push kalyanvasu08/netflix:latest"
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image kalyanvasu08/netflix:latest > trivyimage.txt"
            }
        }
        stage ("Docker Scout Image Analysis ") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh 'docker-scout quickview kalyanvasu08/netflix:latest'
                        sh 'docker-scout cves kalyanvasu08/netflix:latest'
                        sh 'docker-scout recommendations kalyanvasu08/netflix:latest'
                    }
                }
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8000:80 kalyanvasu08/netflix:latest'
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'xxxxxxx@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
