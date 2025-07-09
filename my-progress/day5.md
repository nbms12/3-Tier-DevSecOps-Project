

Architectural diagram 

![image](https://github.com/user-attachments/assets/633b1e62-5596-438e-b5db-77617559cbe0)

steps:

1. create a iam user wit permission to set of policies to create eks and associated resoources

2. install terraform , create resources under aws cloud .
   refered : https://github.com/nbms12/Microservices-Terraform.git


3. install jenkins and sonarqube servers

4. install kubectl , awscli , and eksctl and configure aws 

```
kubectl create namespace -n dev

```
5. inside jenkins server create ServiceAccount, roles , cluster bind,ClusterRole , ClusterRolebind permissions ( refer RBAC folder )  all tis permissions are under dev namesapce


   
eks cluster 

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
