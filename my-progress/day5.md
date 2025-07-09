

Architectural diagram 

![image](https://github.com/user-attachments/assets/633b1e62-5596-438e-b5db-77617559cbe0)

steps:

1. create a iam user wit permission to set of policies to create eks and associated resoources

2. install terraform , create resources under aws cloud .
   
   refered : https://github.com/nbms12/Microservices-Terraform.git

   application source : https://github.com/jaiswaladi246/3-Tier-DevSecOps-Mega-Project


4. install jenkins and sonarqube servers

5. install kubectl , awscli , and eksctl and configure aws 

```
kubectl create namespace -n dev

```
5. inside jenkins server create ServiceAccount, roles , cluster bind,ClusterRole , ClusterRolebind permissions ( refer RBAC folder )  all tis permissions are under dev namesapce

deploy 

```

kubectl apply -f serviceaccount.yaml
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
kubectl apply -f clusterrole.yaml
kubectl apply -f clusterrolebinding.yaml

```

7. setup and configure jenkins  and install  imp pluggins

   .docker
   .kubernetes cli
   .nodejs
   . pipeline stage view


8. create a token from sonarqube

9. add credentails : docker hub cred , sonarqube token , k8s secret value

10. create pipeline job

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
         
         
        
        
            
    }
}

```

   
eks cluster created from terraform 

![image](https://github.com/user-attachments/assets/91006459-c48e-4986-8cde-bff83ccaaa61)

jenkins pipelines view 

![image](https://github.com/user-attachments/assets/72dac41b-48aa-4226-8f1a-4fe0a4bb9ecf)



sonarqube project overview : 

![image](https://github.com/user-attachments/assets/81dd997c-5c47-489e-aba7-20f7f85b5b11)


frontend : 

![image](https://github.com/user-attachments/assets/8962cc9a-749f-4fee-b7b7-eb2048816638)


verify deployments

![image](https://github.com/user-attachments/assets/65cc6bda-aa02-45ef-bbed-34a5775f2173)

verify services :

![image](https://github.com/user-attachments/assets/04e990fa-eaf2-4729-90ce-660777a71926)


verify database : 

![image](https://github.com/user-attachments/assets/8c2e904e-f0cc-4b4a-a288-967df5de8fce)
