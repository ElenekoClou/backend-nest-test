pipeline {
    agent any

    environment {
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
        dockerImagePrefix = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = "gcp-registry"
        imageName = "backend-nest-test-csp"
    }

    stages {
        stage("Inicio") {
            steps {
                sh 'echo "Iniciando pipeline"'
            }
        }

        stage("Build y test") {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                }
            }

            stages {
                stage("Instalación de dependencias") {
                    steps {
                        sh 'npm ci'
                    }
                }

                stage("Ejecución de pruebas") {
                    steps {
                        sh 'npm run test'
                    }
                }

                stage("Construcción de aplicación") {
                    steps {
                        sh 'npm run build'
                    }
                }
            }
        }

        stage("Build y push de imagen Docker") {
            steps {
                script {
                    docker.withRegistry("${registry}", registryCredentials) {
                        sh "docker build -t ${imageName} ."
                        sh "docker tag ${imageName} ${dockerImagePrefix}/${imageName}"
                        sh "docker tag ${imageName} ${dockerImagePrefix}/${imageName}:${BUILD_NUMBER}"
                        sh "docker push ${dockerImagePrefix}/${imageName}"
                        sh "docker push ${dockerImagePrefix}/${imageName}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage("Actualización de Kubernetes") {
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }

            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']) {
                    sh "kubectl -n lab-csp set image deployment/${imageName} ${imageName}=${dockerImagePrefix}/${imageName}:${BUILD_NUMBER}"
                }
            }
        }
    }
}