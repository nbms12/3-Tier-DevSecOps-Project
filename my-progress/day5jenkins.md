```


pipeline {
    agent any
    
    tools {
        nodejs 'nodejs23'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'deploy-to-dev-k8', url: 'https://github.com/jaiswaladi246/3-Tier-DevSecOps-Mega-Project.git'
            }
        }
        
        stage('Frontend Compilation') {
            steps {
                dir('client') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('Backend Compilation') {
            steps {
                dir('api') {
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('GitLeaks Scan') {
            steps {
                sh 'gitleaks detect --source ./client --exit-code 1'
                sh 'gitleaks detect --source ./api --exit-code 1'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=NodeJS-Project \
                            -Dsonar.projectKey=NodeJS-Project '''
                }
            }
        }
        // stage('Quality Gate Check') {
        //     steps {
        //         timeout(time: 1, unit: 'HOURS') {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        //         }
        //     }
        // }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        
        stage('Build-Tag & Push Backend Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('api') {
                            sh 'docker build -t manju811/backend:latest .'
                            sh 'trivy image --format table -o backend-image-report.html manju811/backend:latest '
                            sh 'docker push manju811/backend:latest'
                           
                        }
                    }
                }
            }
        }  
            
        stage('Build-Tag & Push Frontend Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('client') {
                            sh 'docker build -t manju811/frontend:latest .'
                            sh 'trivy image --format table -o frontend-image-report.html manju811/frontend:latest '
                            sh 'docker push manju811/frontend:latest'
                        }
                    }
                }
            }
             
        }  
           stage('deploy to k8s'){
               steps{
                   script{
                       withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'dev', restrictKubeConfigAccess: false, serverUrl: 'https://C2D28377F4F69A37B6A18788C126C6F9.gr7.ap-south-1.eks.amazonaws.com') {
                        // some block
                        sh 'kubectl apply -f k8s/sc.yaml -n dev'
                        sh 'kubectl apply -f k8s/mysql.yaml -n dev'
                        sh 'kubectl apply -f k8s/backend.yaml -n dev'
                        sh 'kubectl apply -f k8s/frontend.yaml -n dev'
                        sleep 25
                        
                      }
                   }
               }
           } 
           stage('verify deployments..') {
               steps{
                   script{
                       withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'dev', restrictKubeConfigAccess: false, serverUrl: 'https://C2D28377F4F69A37B6A18788C126C6F9.gr7.ap-south-1.eks.amazonaws.com') {
                       // some block
                       sh 'kubectl get pods -n dev'
                       sh 'kubectl get svc -n dev'
                      }
                   }
               }
           }
        // stage('K8-deploy') {
        //     steps {
        //         script {
        //             withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'dev', restrictKubeConfigAccess: false, serverUrl: 'https://042CDFB283295491DDF7A7A422346F3B.gr7.ap-south-1.eks.amazonaws.com') {
        //                     sh 'kubectl apply -f k8s/sc.yaml -n dev'
        //                     sh 'kubectl apply -f k8s/mysql.yaml -n dev'
        //                     sh 'kubectl apply -f k8s/backend.yaml -n dev'
        //                     sh 'kubectl apply -f k8s/frontend.yaml -n dev'
        //                     sleep 30
        //                 }
        //         }
        //     }
        // }
        
        
        
            
    }
}


```
