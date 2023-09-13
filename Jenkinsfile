pipeline {
    agent {
        docker {
            image 'chetanachetule/maven-plus-docker' // Use a Maven and Docker image
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket to access host's Docker daemon
        }
    }
    environment {
        SONAR_URL = "http://sonarqube-server:9000" // Update with your SonarQube server URL
        DOCKER_IMAGE = "your-dockerhub-username/project-name:${BUILD_NUMBER}"
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.host.url=${https://43.204.28.255:9000} -Dsonar.login=${SONAR_AUTH_TOKEN}"
                }
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }
        stage('GitHub Update') {
            steps {
                script {
                    // Replace with your GitHub repository details
                    def GIT_REPO_NAME = "your/repo"
                    def GIT_USER_NAME = "your-username"
                    def GIT_BRANCH = "main" // Replace with your branch

                    // Configure Git
                    git branch: GIT_BRANCH, credentialsId: 'github-credentials', url: "https://github.com/${GIT_REPO_NAME}.git"

                    // Update deployment file (e.g., Kubernetes YAML)
                    sh "sed -i 's#image:.*#image: ${DOCKER_IMAGE}#' deployment.yaml"
                    
                    // Commit and push changes to GitHub
                    sh """
                        git config user.email "your-email@example.com"
                        git config user.name "${GIT_USER_NAME}"
                        git add deployment.yaml
                        git commit -m "Update Docker image tag"
                        git push origin ${GIT_BRANCH}
                    """
                }
            }
        }
    }
}
