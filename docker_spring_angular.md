# Update Angular-Spring app 

## Continuous Deployment using Docker 

Below are the changes required in the build step for pipeline_spring and angularbuild project - 


* pipeline_spring 

1. Add a new parameter 

2. Select **This Project is parameterized  -->  String Parameter** 

3. name: frontendip

4. Default Value - **External IP address of your VM** 

5. Modify the pipeline script as below - 

```
node ('slave1') {
    
    stage('codecheckout') {
   git 'https://github.com/hub-kubernetes/jenkins-cicd.git'
}
   

    dir('maven_integration_demo/spring-boot-samples/spring-boot-angular') {
         stage('updateipaddress') {
             sh label: '', script: 'sed -i "s/SPRING_SERVER/${frontendip}/g" src/main/java/com/baeldung/application/controllers/UserController.java'
         }
         
         stage('buildcode') {
             sh label: '', script: 'mvn clean compile test package'
         }
          stage('createartifact') {
             archiveArtifacts 'target/*.jar'
          }
          stage ('dockerbuild') {
              sh label: '', script: 'cp dockerfile_java/Dockerfile .'
              sh label: '', script: 'docker build . -t springjavabackend'
          }
          stage ('deploycontainer'){
              sh label: '', script: 'docker run -d -p 8888:8080 springjavabackend'
          }
}
}
```

* angularbuild 

Modify the execute shell script as below - 

```

cd maven_integration_demo/spring-boot-samples/spring-boot-angular/angularclient/src/app/service

sed -i "s/SPRING_BACKEND/${spring_server}/g" user.service.ts

sed -i "s/SPRING_PORT/${spring_port}/g" user.service.ts

#Go back to Build directory to execute npm install

cd ../../../

# Install angular Build

npm i rxjs-compat

npm install

npm run build

cp ../dockerfile_angular/Dockerfile . 

docker build . -t angularfe 

docker run -d -p 4200:80 angularfe
```


Log in to the server on which you have jenkins installed 

```
sudo -i 

usermod -aG docker jenkins 
```
