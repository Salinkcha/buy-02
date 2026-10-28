pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    tools {
        maven 'Maven3'
        nodejs 'NodeJS'
    }

    environment {
        JWT_SECRET = credentials('jwt-secret')
        INTERNAL_TOKEN = credentials('internal-token')
        MAVEN_OPTS = "-Xmx512m -XX:MaxMetaspaceSize=256m"
    }

    parameters {
        booleanParam(name: 'ROLLBACK', defaultValue: false, description: 'Déclencher un rollback manuel (sans build)')
    }

    stages {
        
        stage('Checkout Git') {
            when { expression { params.ROLLBACK == false } }
            steps {
                sh 'docker network connect buy02-runtime_buy-net buy02-runtime-jenkins-1 || true'
                echo 'Git Checkout in Progress...'
                checkout scm
                sh 'ls -la'
                sh 'ls -la backend/ || true'
            }
        }

        stage('Docker Check') {
            when { expression { params.ROLLBACK == false } }
            steps {
                sh 'whoami'
                sh 'id'
                sh 'docker version'
                sh 'docker compose version'
                sh 'docker ps'
            }
        }

        stage('Start MongoDB') {
            when { expression { params.ROLLBACK == false } }
            steps {
                script {
                    echo 'Starting MongoDB...'
                    sh 'docker compose -p buy02-runtime up -d mongo'
                    sh 'docker ps | grep mongo || (echo "MongoDB failed to start" && exit 1)'
                }
            }
        }
        
        stage('Build & Test Backend') {
            when { expression { params.ROLLBACK == false } }
            parallel {
                stage('User Service Test') {
                    steps {
                        dir('backend/user-service') {
                            sh 'mvn clean test -DforkCount=1 -DreuseForks=false'
                        }
                    }
                }
                stage('Product Service Test') {
                    steps {
                        dir('backend/product-service') {
                            sh 'mvn clean test -DforkCount=1 -DreuseForks=false'
                        }
                    }
                }
                stage('Media Service Test') {
                    steps {
                        dir('backend/media-service') {
                            sh 'mvn clean test -DforkCount=1 -DreuseForks=false'
                        }
                    }
                }
                stage('Order Service Test') {
                    steps {
                        dir('backend/order-service') {
                            sh 'mvn clean test -DforkCount=1 -DreuseForks=false'
                        }
                    }
                }
                stage('Cart Service Test') {
                    steps {
                        dir('backend/cart-service') {
                            sh 'mvn clean test -DforkCount=1 -DreuseForks=false'
                        }
                    }
                }
            }
        }

        stage('SonarQube Scan') {
            when { expression { params.ROLLBACK == false } }

            parallel {
                stage('User Service Scan') {
                    steps {
                        dir('backend/user-service') {
                            withSonarQubeEnv('sonarqube') {
                                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.projectKey=buy-02-user -Dsonar.projectName="buy-02-user" -Djava.net.preferIPv4Stack=true -Dsonar.exclusions=**/target/**,**/node_modules/**,**/*.spec.ts,**/generated-sources/** -Dsonar.java.binaries=target/classes'
                            }
                        }
                    }
                }
                stage('Product Service Scan') {
                    steps {
                        dir('backend/product-service') {
                            withSonarQubeEnv('sonarqube') {
                                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.projectKey=buy-02-product -Dsonar.projectName="buy-02-product" -Djava.net.preferIPv4Stack=true -Dsonar.exclusions=**/target/**,**/node_modules/**,**/*.spec.ts,**/generated-sources/** -Dsonar.java.binaries=target/classes'
                            }
                        }
                    }
                }
                stage('Media Service Scan') {
                    steps {
                        dir('backend/media-service') {
                            withSonarQubeEnv('sonarqube') {
                                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.projectKey=buy-02-media -Dsonar.projectName="buy-02-media" -Djava.net.preferIPv4Stack=true -Dsonar.exclusions=**/target/**,**/node_modules/**,**/*.spec.ts,**/generated-sources/** -Dsonar.java.binaries=target/classes'
                            }
                        }
                    }
                }
                stage('Order Service Scan') {
                    steps {
                        dir('backend/order-service') {
                            withSonarQubeEnv('sonarqube') {
                                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.projectKey=buy-02-order -Dsonar.projectName="buy-02-order" -Djava.net.preferIPv4Stack=true -Dsonar.exclusions=**/target/**,**/node_modules/**,**/*.spec.ts,**/generated-sources/** -Dsonar.java.binaries=target/classes'
                            }
                        }
                    }
                }
                stage('Cart Service Scan') {
                    steps {
                        dir('backend/cart-service') {
                            withSonarQubeEnv('sonarqube') {
                                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.projectKey=buy-02-cart -Dsonar.projectName="buy-02-cart" -Djava.net.preferIPv4Stack=true -Dsonar.exclusions=**/target/**,**/node_modules/**,**/*.spec.ts,**/generated-sources/** -Dsonar.java.binaries=target/classes'
                            }
                        }
                    }
                }
            }
        }

        stage('SonarQube Quality Gates') {
            when { expression { params.ROLLBACK == false } }

            steps {
                dir('backend/user-service') {
                    script { env.CURRENT_SERVICE = 'User Service' }
                    waitForQualityGate(abortPipeline: true)
                }
                dir('backend/product-service') {
                    script { env.CURRENT_SERVICE = 'Product Service' }
                    waitForQualityGate(abortPipeline: true)
                }
                dir('backend/media-service') {
                    script { env.CURRENT_SERVICE = 'Media Service' }
                    waitForQualityGate(abortPipeline: true)
                }
                dir('backend/order-service') {
                    script { env.CURRENT_SERVICE = 'Order Service' }
                    waitForQualityGate(abortPipeline: true)
                }
                dir('backend/cart-service') {
                    script { env.CURRENT_SERVICE = 'Cart Service' }
                    waitForQualityGate(abortPipeline: true)
                }
            }
        }
        
        stage('Build & Test Frontend') {
            when { expression { params.ROLLBACK == false } }
            steps {
                dir('frontend') {
                    sh 'PUPPETEER_SKIP_DOWNLOAD=true npm ci --no-audit --no-fund'
                    sh 'CHROME_BIN_PATH="$(command -v chromium || command -v chromium-browser)" && test -x "$CHROME_BIN_PATH"'
                    sh 'npm run build'
                    sh 'export CI=true CHROME_BIN="$(command -v chromium || command -v chromium-browser)" && npm test -- --project=buy-frontend --code-coverage --no-watch'
                    withSonarQubeEnv('sonarqube') {
                        sh 'npx sonarqube-scanner -Dsonar.projectKey=buy-02-front -Dsonar.projectName="buy-02-front" -Dsonar.sources=src -Dsonar.exclusions=**/node_modules/**,**/*.spec.ts -Dsonar.javascript.lcov.reportPaths=coverage/buy-frontend/lcov.info'
                    }
                    waitForQualityGate(abortPipeline: true)
                }
            }
        }
    
        stage('Deploy with Rollback Strategy') {
            when { expression { params.ROLLBACK == false } }
            steps {
                script {
                    echo '🚀 Starting deployment process...'
                    
                    sh 'docker images --format "{{.Repository}}:{{.Tag}}" | grep buy02-runtime | grep \':latest$\' > /tmp/current_images.txt || true'
                    sh '''
                    for img in $(cat /tmp/current_images.txt); do
                        base_img=$(echo "$img" | cut -d':' -f1)
                        docker rmi "${base_img}:latest-backup" 2>/dev/null || true
                        docker tag "$img" "${base_img}:latest-backup"
                    done
                    '''
                    
                    try {
                        sh 'docker compose -p buy02-runtime build frontend user-service product-service media-service order-service cart-service'
                        sh 'docker compose -p buy02-runtime up -d --force-recreate frontend user-service product-service media-service order-service cart-service'
                        sh 'echo "Waiting for services to stabilize..." && sleep 10'
                        
                    } catch (Exception e) {
                        echo '❌ Error detected, rollback starting...'
                        sh '''
                            for service in frontend user-service product-service media-service order-service cart-service; do
                                if docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "buy02-runtime-${service}:latest-backup"; then
                                    docker tag buy02-runtime-${service}:latest-backup buy02-runtime-${service}:latest
                                    echo "Restored buy02-runtime-${service}"
                                fi
                            done
                            docker compose -p buy02-runtime up -d --force-recreate frontend user-service product-service media-service order-service cart-service
                        '''
                        error('Deployment failed and rollback executed. Check logs for details.')
                    }
                }
            }
        }

        stage('Manual Rollback') {
            when { expression { params.ROLLBACK == true } }
            steps {
                script {
                    echo 'Manual rollback triggered...'
                    sh '''
                        for service in frontend user-service product-service media-service order-service cart-service; do
                            if docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "buy02-runtime-${service}:latest-backup"; then
                                docker tag buy02-runtime-${service}:latest-backup buy02-runtime-${service}:latest
                                echo "Restored buy02-runtime-${service}"

                            fi
                        done
                        docker compose -p buy02-runtime up -d --force-recreate frontend user-service product-service media-service order-service cart-service
                    '''
                    echo '✅ Rollback completed. Services should be restored to previous stable versions.'
                }
            }
        }
    }
    
    post {
        always {
            echo "📊 Pipeline execution finished. Status: ${currentBuild.currentResult}"
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
        success {
            echo '🎉 All stages completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed! Check logs above.'
        }
    }
}