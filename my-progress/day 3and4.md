day 3 , 4 

output : 
create 2 instances : 

![image](https://github.com/user-attachments/assets/2dc65223-4b68-413d-9784-98e0fa00df80)


add ports inside jenkins server , 

![image](https://github.com/user-attachments/assets/62093d87-cf5f-4f9c-be62-d5084d5caf7c)


pipeline yml file 

```
pipeline {
    agent any
    tools {
        nodejs 'nodejs23'
    }
    environment {
        SCANNER_HOME = tool 'sonarscanner'
    }
    stages {
        stage('git checkout') {
            steps {
                git branch: 'docker-build-deploy', url: 'https://github.com/jaiswaladi246/3-Tier-DevSecOps-Mega-Project.git'
            }
        }
        stage('fronend-cmpile') {
            steps {
               dir('client') {
                 sh 'find . -name "*.js" -exec node --check {} +'
               }
            }
        }
        stage('backend -compile') {
            steps {
                dir('api') {
                 sh 'find . -name "*.js" -exec node --check {} +'
               }
            }
        }
         stage('git leaks scan') {
            steps {
                sh 'gitleaks detect --source ./client --exit-code 100'
                sh 'gitleaks detect --source ./api --exit-code 100'
            }
        }
        
        // stage('SCANNER-QUALITY-SCORE') {
        //     steps {
        //         //git branch: 'dev', url: 'https://github.com/jaiswaladi246/3-Tier-DevSecOps-Mega-Project.git'
        //          timeout(time: 1, unit: 'HOURS') {
        //           // some block
        //           waitForQualityGate abortPipeline: false, credentialsId: 'sonarcred'
        //       }
        //     }
        // }
        stage('trivy scan') {
            steps {
                sh 'trivy fs --format table -o myreport.html .'
            }
        }
        stage('build docker  for backend api ') {
            steps {
                 script{
                     withDockerRegistry(credentialsId: 'docker-cred') {
                      dir('api') {
                          sh 'docker rmi manju811/backend:latest || true'
                          sh 'docker build -t manju811/backend:latest .'
                          sh 'trivy image --format table -o myreportapi.html manju811/backend:latest'
                          sh 'docker push manju811/backend:latest'
                          
                        }
                     }
                 }
            }
        }
        stage('build docker  for frontend-user ') {
            steps {
                 script{
                     withDockerRegistry(credentialsId: 'docker-cred') {
                      dir('api') {
                          sh 'docker rmi manju811/frontend:latest || true'
                          sh 'docker  build -t manju811/frontend:latest .'
                          sh 'trivy image --format table -o myreportfrontend.html manju811/frontend:latest'
                          sh 'docker push manju811/frontend:latest'
                        }
                     }
                 }
            }
        }
         stage('docker deploy via docker compose  ') {
            steps {
                 script{
                     sh 'docker-compose up -d'
                 }
            }
         }
    } 
}

```

jenkins stages view 

![image](https://github.com/user-attachments/assets/41422369-3de5-443e-ba0e-6eedfbca1df1)


check contianers run status 

![image](https://github.com/user-attachments/assets/96213bd7-0ef0-4b2a-be67-0d48008ad6c1)



![image](https://github.com/user-attachments/assets/b376aa70-588e-43ef-9212-20bb6cba86aa)
