@Library('Shared') _

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = 'nithesh06/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'nithesh06/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github-credentials')
        GIT_BRANCH = "master"
    }
    
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                script {
                    clone("https://github.com/Nithesh0602/tws-e-commerce-app_hackathon.git","master")
                }
            }
        }

stage('Sync NPM Lockfile') {
    steps {
        script {
            docker.image('node:18-alpine').inside {
                // Install git inside the container
                sh 'apk add --no-cache git'

                // Create a writable cache directory for npm
                sh 'mkdir -p /var/lib/jenkins/workspace/easyshop/.npm-cache'

                // Run npm install with custom cache
                sh 'export NPM_CONFIG_CACHE=/var/lib/jenkins/workspace/easyshop/.npm-cache && npm install'

                // Configure Git identity
                sh 'git config user.email "ci-bot@example.com"'
                sh 'git config user.name "CI Bot"'

                // Commit and push updated lockfile
                sh 'git add package-lock.json'
                sh 'git commit -m "chore: sync package-lock.json with package.json" || echo "No changes to commit"'
                sh 'git push origin ${env.GIT_BRANCH}'
            }
        }
    }
}


        
        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }
                
                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }
        
        stage('Security Scan with Trivy') {
            steps {
                script {
                    trivy_scan()
                }
            }
        }
        
        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
                
                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'Nithesh',
                        gitUserEmail: 'kpnitheshmech@gmail.com'
                    )
                }
            }
        }
    }
}
