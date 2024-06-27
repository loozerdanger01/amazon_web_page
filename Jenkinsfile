pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/loozerdanger01/amazon_web_page.git'
            }
        }
        stage("Sonarqube-scanner"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon \
                    -Dsonar.projectKey=Amazon '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dpcheck'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t amazon ."
                       sh "docker tag amazon nithi309/amazon:latest "
                       sh "docker push nithi309/amazon:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nithi309/amazon:latest > trivyimage.txt"
            }
        }
        stage('Deploy'){
            steps{
                sh 'docker run -d --name amazon -p 3000:3000 nithi309/amazon:latest'
            }
        }
    }
}
