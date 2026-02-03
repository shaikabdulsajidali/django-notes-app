# Jenkins Learning
### Jenkins Installation
1. Login AWS
2. create EC2 instance of linux os
3. Add security groups
   - http
   - https
   - ssh
   - 8080
4. Go to 'https://www.jenkins.io/doc/book/installing/linux/' in browser for installtion
   - install openjdk 
   - install jenkins
5.  launch app
     `<public_ec2_ip>:8080`
6. set it up jenkins
   

### Agent (instance)
1. Login AWS
2. create EC2 instance of linux os
3. Add security groups
   - http
   - https
   - ssh
   - 8000
4. Go to 'https://www.jenkins.io/doc/book/installing/linux/' in browser for installtion
   - install openjdk 
5. install docker in instance
   ```
   sudo apt-get install docker.io
   sudo usermod -aG docker $USER && newgrp docker
   ```
6. install docker compose in instance
  ```
  sudo apt install docker-compose-v2
  docker compose version
  ```
6. Copy paste your controller instance public key in ssh
   ```
   cd ~/.ssh
   vim authorized_keys
   ```
7. Append your public key in authorized_keys
   

### Agent creation
1. Login jenkins
2. navigate to `Manage Jenkins` in that `Nodes` in that `new node` in that it will display below :
   - Name : Agent_name
   - Description : Enter something in desc
   - Number of executors : 1 (any)
   - Remote root directory : /home/ubuntu
   - Labels : label_name (any name) (important)
   - Usage
   - Launch method
     - Launch agents via ssh
       - Host : Agent_instance_Ip
       - Credentials : if available select or create
       - Host Key Verification Strategy :
   - Availability
   - Save


NOTE:
- Always launcch your agent first
- Copy paste public key from controller to agent


### pipeline creation
1. Jenkins Dashboard
2. new item
   - name of agent
   - Type of item : Pipieline
   - ok
3. General
    - Description : any
    - Github project
      - project url
4. Triggers
     - GitHub hook trigger for GITScm polling
5. Pipeline
     - Definition
     - script (Grrovy)
    ```
    pipeline {
        agent {
            label "sam"
        }
    
        stages {
            stage('Code') {
                steps {
                    echo 'in this stage Cloning repo'
                    git url:"https://github.com/shaikabdulsajidali/django-notes-app.git", branch: "main"
                    echo "cloning successfull"
                }
            }
            stage('build') {
                steps {
                    echo 'Building image'
                    sh 'docker build -t notesapp:latest .'
                    echo 'build success'
                }
            }
            stage('test') {
                steps {
                    echo 'This is Testing stage'
                }
            }
            stage('Push to docker hub') {
                steps {
                    echo 'In this stage we push build image to docker hub'
                    withCredentials([usernamePassword(
                        credentialsId:'DockerHubCred',
                        passwordVariable:'DockerHubPass',
                        usernameVariable:'DockerHubUser'
                    )]){
                        sh '''
                            docker login -u $DockerHubUser -p $DockerHubPass
                            docker image tag notesapp:latest $DockerHubUser/notesapp:latest
                            docker push $DockerHubUser/notesapp:latest
                        '''
                    }
                    echo 'Push successfull'
                }
            }
            stage('deploy') {
                steps {
                    echo 'deploy stage'
                    sh 'docker compose up -d --build'
                    echo "deployment successful"
                }
            }
        }
    }
    ```
6. Save
