pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = "kushagrag99/netflix-app:v1.${BUILD_NUMBER}"
       REGISTRY_CREDENTIALS = credentials('docker-cred')
       GIT_REPO_NAME = "devsecops-netflix-jenkins-k8s"
       GIT_USER_NAME = "kushagra-g9"
       KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-jenkins' // Jenkins credential ID for kubeconfig
       PATH = "${env.PATH}:/usr/local/bin"
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/kushagra-g9/devsecops-netflix-jenkins-k8s.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix-test '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube' 
                }
            } 
        }
<<<<<<< HEAD
         }
=======
        }
>>>>>>> 3487eefd507d7614b3c1d9ba1847802916ae9748
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
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
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=<yourtmdbapikey> -t netflix ."
                       sh "docker tag netflix ${DOCKER_IMAGE}"
                       sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image ${DOCKER_IMAGE} > trivyimage.txt" 
            }
        }
      
      stage('Update Deployment File') {
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          sh '''
            git config --global user.email "guptakushagra99@gmail.com"
            git config --global user.name "Kushagra Gupta"
            sed -i "s|\\(kushagrag99/netflix-app:\\).*|\\1v1.${BUILD_NUMBER}|" Kubernetes-manifests/deployment.yml
            git add Kubernetes-manifests/deployment.yml
            git commit -m "Update deployment image to version v1.${BUILD_NUMBER}" || echo "No changes to commit"
            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }

        stage('Deploy to kubernetes'){
            steps{
                script{
                    dir('Kubernetes-manifests') {
                       withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG_FILE')]) {
                               sh '''
                                  export KUBECONFIG=$KUBECONFIG_FILE
                                 kubectl apply -f deployment.yml
                                 kubectl apply -f service.yml
                                 '''
                        }   
                    }
                }
            }
        }

    }
    


