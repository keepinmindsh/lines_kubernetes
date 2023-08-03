# Exam

- swit-dev gcp / gke 환경 연결하기 
- {your-name}은 본인의 이니셜로 작성한다.
- 본인이 사용할 Image 하나를 만들어서 사전에 GCR에 업로드하여 사용하거나, 흔히 시험에 사용하는 busybox 또는 nginx를 사용해도 무방함.   
- 본인의 github 계정에 각 exam 폴더를 생성한다.
  - 예시) exam/파일,업로드 자료
  - exam 별로 진행하는 Shell Script를 ReadMe.md에 작성하여 제출한다.

```
# Exam1

~~ 처리내용 기입 

# Exam2

~ 처리내용 기입
```

최종을 public github 경로에 push 하기 

- 각 문제별로 출력해야하는 결과를 Script 내에 작성하여 제출 한다. 
- 생성된 모든 파일 및 결과를 각 exam 폴더 내에서 추가할 것 

#### Exam 1
Get the list of nodes in JSON format and store it in a file at {your-folder}/json-{your-name}.json

#### Exam 2
Create Namespace exam-{your-name}

#### Exam 3 
Create a new deployment {your-name}-deployment --image={your-image} --replicas=2 of namespaces exam-{your-name}

#### Exam 4
Print the names of all deployments in the exam-{your_name} namespace in the following format:   

DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE  

#### Exam 5
Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. 
Next upgrade the deployment to version 1.17 using rolling update. 
Make sure there version update is recorded int the resource annotation.

#### Exam 6
Create a new service "web-application" :  
Name: web-application; Type: NodePort; port: 8080; nodePort: 30083; selector: simple-webapp

#### Exam 7
Create a service {your-name}-service to expose the messaging application within the cluster on port 9090

#### Exam 8
- 1. Create a deployment named {your-name}-deployment using a pod {your-name}-pod of the image {your-image} with 2 replicas with yaml.
- 2. Change Replica count from 2 to 1 at yaml file and update your original deployment.
- 3. Setting config-map as a volume of pod
config-map key=value    

hello=world
drink=good
happy=work

#### Exam 9
Access your pod and make a file and print "Hello World! to "hello.txt" files. 

#### Exam 10 
Create a pod myapp-pod and the use an initContainer that uses the busybox image and sleeps for 20 seconds 
