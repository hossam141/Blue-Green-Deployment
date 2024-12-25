pipeline {
    agent any 
    tools{
        maven 'maven3'    
    }

    parameters {
    choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
    choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
    booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
    }


    environment {
        IMAGE_NAME = "hossamtaha18/bankapp"
        TAG = "${params.DOCKER_TAG}"
        SCANNER_HOME = tool 'sonnar-scanner'
    }

    stages{
        stage ('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/hossam141/Blue-Green-Deployment.git'
            }

        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('Trivy FS Scan'){
            steps{
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('sonar'){
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projecKey=Multitier -Dsonar.projectName=Multitier -Dsonar.java.binaries=target"
                }
            }
        }

        stage('Quality Gate Check'){
            steps{
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Buid'){
            steps{
                sh "mvn package -DskipTests=true"
            }
        }

        stage('Publish Artifact to Nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy-DskipTests=true"
                }
            }
        }

        stage('Docker Build and Tag Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                        sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    }
                }
            }
        }

        stage('Docker Build and Tag Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                        sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs.html ${IMAGE_NAME}:${TAG}"
            }   
        }          

        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push ${IMAGE_NAME}:${TAG}"
                    }
                }
            }
        }
    }
}