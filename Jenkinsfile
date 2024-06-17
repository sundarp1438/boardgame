pipeline{
    agent{
        label 'rocky_9'
    }
    tools{
        jdk 'JAVA_HOME'
        maven 'MAVEN_HOME'
    }
    environment {
        SCANNER_HOME=tool 'sonar-server'
    }
    stages{
        stage('Workspace Cleaning'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/sundarp1438/boardgame.git'
            }
        }
        stage('compile source code'){
            steps{
                
                sh '''echo compiling the source code
                mvn compile'''
            }
        }
        stage('test compile source code'){
            steps{
                sh '''echo testing the compiled source code using suitable unit testing framework
                mvn test''' 
            }
        }
        stage('build project'){
            steps{
                sh '''echo cleaing the maven projct by deleting any existing target directory and build the project
                mvn clean package
                ls -al target'''
            }
        }
        stage('scan file system'){
            steps{
                sh '''echo scanning the files in the cloned git repository using trivy
                trivy fs --format table -o trivy-fs-report.html .'''
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Board-Game \
                    -Dsonar.projectKey=Board-Game \
                    -Dsonar.exclusions=**/*.java
                    '''
                }
            }
        }
        stage("Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        
        stage('deploy the artifact to nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: '', jdk: 'JAVA_HOME', maven: 'MAVEN_HOME', mavenSettingsConfig: 'nexus', traceability: true) {
                    sh '''echo deploying the build artifact to the nexus repository with version 0.0.${BUILD_NUMBER}
                    mvn deploy'''
                }
            }
        }
        stage('OWASP DP SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    sh 'docker build -t sundarp1985/buildgame-app-pipeline:latest .'
            }
        }
    }
        stage('Containerize And Test') {
            steps {
                script{
                    sh 'docker run -d --name buildgame_app sundarp1985/buildgame-app-pipeline:latest && sleep 10 && docker stop buildgame_app'
                }
            }
        }
        stage('Push Image To Dockerhub') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubPass')]) {
                    sh 'docker login -u sundarp1985 --password ${DockerHubPass}' }
                    sh 'docker push sundarp1985/buildgame-app-pipeline:latest'
                }
            }
        }  
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image sundarp1985/buildgame-app-pipeline:latest > trivyimage.txt" 
            }
        }
        
    //    stage('Deploy to Kubernetes'){
    //         steps{
    //             script{
    //                 dir('Kubernetes') {
    //                     withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
    //                             sh 'kubectl apply -f deployment-service.yml'
    //                             sh 'kubectl get svc'
    //                             sh 'kubectl get all'
    //                     }   
    //                 }
    //             }
    //         }
        
    // }
}
}
