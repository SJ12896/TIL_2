# DevOps Project - 2022: CI/CD with Jenkins Ansible Kubernetes

## 1. Introduction

1. build and deploy application on Tomcat server

   - going to set up ci/cd pipeline with githubm jenkins, maven, tomcat
   - jenkins server 세팅으로 시작해서 maven, git set up
   - tomcat server set up
   - github, maven, tomcat server를 jenkins와 통합
   - ci/cd jenkins job을 생성하고 build해서 tomcat server에 배포

2. test deployment

   ![image-20220914105624773](C:\Users\sj\AppData\Roaming\Typora\typora-user-images\image-20220914105624773.png)

3. deploy on a Dockekr container

   - set up docker enviornment
   - wirte dockerfile
   - 도커 호스트 젠킨스와 통합(이미지와 컨테이너 만들기)
   - build and deploy 위해 jenkins ci/cd job 만들기
   - vm대신 docker container에 deploy

4. deploy on a container with ansible

   - setup ansible server
   - docker host를 ansible과 통합
   - ansible playbook 이미지, 컨테이너 만들어 jenkins와 통합
   - ansible에 build하고 docker deploy위해 create ci/cd job

5. ![image-20220914110442176](C:\Users\sj\AppData\Roaming\Typora\typora-user-images\image-20220914110442176.png)

6. build and deploy on kubernetes

   - set up k8s on aws(EKS - Elastic Kubernetes Service)
   - write pod, service, deployment manifest files
   - intergrating k8s with ansible
   - create ci/cd job to build code on ansible to deploy it on an k8s cluster.

   ![image-20220914110844872](C:\Users\sj\AppData\Roaming\Typora\typora-user-images\image-20220914110844872.png)



- Continuous Integration (CI)
- Continuous Delivery (CD)
- Continuous Deployment (CD)
- 로컬에서 작성한 코드를 소스 코드 관리 시스템에 올리면 CI툴이 자동으로 코드를 받고 빌드해서 유닛테스트를 실행한다. CI과정에서 소스 코드를 빌드하고 컴파일해서 artifacts를 만들어야 한다. 그리고 원하는 환경에 배포한다. 그리고 회귀 테스트를 실행한 후 실제 운영환경으로 배포. 이 과정에서 수동적인 개입없이 수행해 Continuous Deployment라고 부른다. 반면 수동적인 개입이 필요하다면 Continuous delivery라고 한다.



## 2. CI/CD pipeline using Git, Jenkins and Maven

- ec2 인스턴스를 생성한 후 public ip에 mobaxterm으로 접근. sudo su - 로 root계정에서 java와 jenkins설치 진행(repository 가져온후 `amazon-linux-extras` install 실행)
- `service jenkins status`로 상태 확인, `service jenkins start`로 실행
- ec2 보안그룹에서 기본 22포트 말고 사용자 지정 TCP를 추가해 포트번호는 8080으로 한다. 그리고 public ip:8080으로 접근하면 unlock jenkins를 볼 수 있다. 기본 비밀번호가 저장된 위치가 나오는데 서버에서 `cat 위치`로 확인한다.
- 새로운 job을 만들어 execute shell을 택하고 

```shell
echo "Hello World"
uptime
```

을 저장하면 빌드했을 때 해당 명령을 실행한다.(uptime은 현재 시간, 부팅 후 경과 시간, 로그인한 유저 수, 시스템 부하율 등을 보여준다.)

- GitHub 플러그인을 추가한 후 해당 레포지토리 url을 사용해 코드 pull
- /opt로 이동해서 `wget {메이븐 다운로드 링크}`로 maven다운로드
- mvn 명령어를 다른 디렉토리에서도 사용가능하도록 .bash_profile에 환경변수 설정. M2_HOME, M2, JAVA_HOME등. JAVA_HOME은 jdk 위치로 정한다. 또 그 외에 위치를 볼 수 있도록 $PATH에 각각의 위치를 적는다.
- jenkins에서 maven 플러그인도 설치
- 각 플러그인들을 설치한 후 global configuration option에서 설정, JAVA_HOME 위치, MAVEN_HOME위치 등 입력해줌.
- java project에 jenkins사용해서 빌드: 새로운 아이템을 만들어 이번엔 maven project를 택한다. 전과 똑같이 git주소 입력하고 이번엔 build에서 goals를 maven lifecycle에 나온 것 중에 하나인 install을 택한다.(앞선 단계 자동 실행됨) `clean install`로 지정. 이제 build now를 하면 코드를 받아온다.



## 3. Integrating Tomcat server in CI/CD pipeline

- EC2 Tomcat_Server 인스턴스 생성. 보안그룹에서 8080포트를 추가한다.
- amazon-linux-extras로 jdk11설치 -> opt로 이동해서 -> wget으로 톰캣링크에서 다운로드 -> tar -xvzf로 압축해제 -> 압축해제 폴더의 bin에서 startup.sh실행하면 tomcat started -> public ip:8080으로 들어가면 톰캣 시작된거 나옴
- manager app 들어가면 403 access denied -> manager는 브라우저가 톰캣이랑 같은 일하는 곳에서 실행되어야 하기 때문에 외부에서 접근하려면 context update필요하다. webapps에 존재하는 context.xml중 examples외에 있는 두 파일을 업데이트한다.
  - 두 파일 다 className RemoteAddrValve의 allow까지 주석처리한다. <!-- -->
- bin에서 shutdown.sh로 종료 후 재실행하면 login이 필요하다.
- conf폴더의 tomcat-users.xml에 role과 user를 추가하면서 username, password를 설정할 수 있다.
- 원할 때 실행하기 위해 tomcat startup.sh와 shutdown.sh를 위한 링크 파일을 생성해서 쉽게 서버를 껐다 재시작하면 로그인이 가능하다.
- deploy to container 플러그인 설치
- jenkins에서 tomcat server를 위한 credential을 추가한다. username with password 방식으로 하는데 tomcat-users.xml에 있던 manager-script role과 동일하다. 
- 그리고 새로운 아이템을 생성한다. BuildAndDeploy라는 이름의 메이븐 프로젝트다. git 주소입력 clean install 지정을 하고 post-build actions를 `deploy war/ear to a container`로 한다. war/ear files는 webapp.war를 정확히 지정해도 디지만 `**/*.war`를 사용하면 일반적이다. containers는 tomcat 버전을 택해 지정하고 credentials를 택한 뒤 tomcat url은 public ip:포트번호로 지정한다.
- 빌드 후 주소/webapp/으로 들어가면 화면이 보인다.
- 코드를 수정 후 github에 푸시한 뒤 jenkins에서 지금 빌드를 클릭하면 수정한 내용이 즉시 반영된다.
- 하지만 깃허브 수정사항을 자동으로 반영하기 위해 빌드 트리거를 설정한다. `configure`로 들어가 build trigger에서 원하는 옵션을 선택한다. 지금은 Poll SCM을 택하고 * * * * *를 사용해 매분 반영되게 한다.



## 4. Integrating Docker in CI/CD Pipeline

- Docker-Host라는 새로운 EC2 인스턴스를 생성한다. 보안그룹은 전에 사용한 것과 동일하다.

- moba x term에서 `yum install docker -y`로 설치 설치후 `service docker start`로 시작
- server이름 바꾸고 싶다면 `vi /etc/hostname`에서 원하는 이름으로 변경하고 `init 6`를 입력하면 서버가 종료된다. 
- `docker pull tomcat`으로 도커 허브에서 톰캣 이미지 받아오기.
- ` docker run -d --name tomcat-container -p 8081:8080 tomcat`로 컨테이너를 실행하는데 앞의 포트 넘버는 외부, 뒤는 내부다.
- 컨테이너 실행 후 포트번호 8081로 접속하면 접속되지 않는데 ec2인스턴스 보안그룹에서 8081을 아직 열지 않았기 때문이다. 인바운드 규칙에 8081-9000을 새로 추가한 뒤 다시 시도한다. 아직 요청하는 페이지를 찾을 수 없어 404에러가 발생한다.
- `docker exec -it tomcat-container/bin/bash`를 사용하면 해당 컨테이너에 bin/bash를 사용해 도커 컨테이너에 접속한다. ls로 살펴보면 webapps와 webapps.dist가 존재한다. webapps에서 내부파일을 살피면  브라우저에 제공해야할 핑리이 아무것도 없고 webapps.dist에 ROOT, docs, examples, host-manager, manager가 존재한다. 이 파일들을 webapps로 복사하거나 이동시켜야 한다. 
- 이 에러는 새 컨테이너를 생성하면 똑같이 발생하므로 dockerfile을 작성해야 한다. 

![image-20220916145125790](C:\Users\sj\AppData\Roaming\Typora\typora-user-images\image-20220916145125790.png)

![image-20220916162410242](C:\Users\sj\AppData\Roaming\Typora\typora-user-images\image-20220916162410242.png)

- 도커파일 작성 후 `docker build -t 컨테이너이름`을 통해 실행하고 docker run으로 실행하는데 이번엔 외부 포트 번호를 8083으로 지정한다.
- vm에서 tomcat을 설치해 실행하는 방법말고 바로 tomcat 이미지를 가져와 실행할 수 있다. 이 경우 앞선 404에러가 발생하므로 webapp.dist를 webapp으로 바꾸는 dockerfile 작성이 필요하다. catalina_home 위치는 dockerhub의 tomcat에 나와있으므로 이 위치를 참고해 파일을 옮긴다.

- Integrate Docker with Jenkins
  - Create a dockeradmin user
    - 존재하는 user확인: `cat /etc/passwd`
    - 유저 그룹 확인: `cat /etc/group`로 확인하면 docker라는 그룹이 존재. 이 그룹에 user를 추가해야 한다.
    - `useradd dockeradmin`
    - 암호 변경: `passwd dockeradmin`
    - `id dockeradmin`으로 id와 속한 그룹 확인
    - `usermod -aG docker dockeradmin`
    - 여기까지 한 후 dockeradmin으로 로그인을 시도하면 server refused our key가 나타난다.
    - 인증을 활성화하기 위해 `vi /etc/ssh/sshd_config`에서 수정
    - /Password를 바로 입력하면 어디에 위치한지 찾을 수 있다. passwordAuthentication  yes는 주석처리되어있기 때문에 해제하고 no를 주석처리 한다.
    - `service sshd reload`로 재시작. stop했다 재시작하면 docker연결이 사라지기 때문에 reload를 한다. 이제 useradmin으로 로그인이 가능.
  - Install `Publish over SSH` plugin
    - manage plugins에서 publish over ssh 설치
    - manage jenkins-configure system에서 publish over ssh 존재. key대신 여기선 SSH Servers를 add해서 인증한다.
    - servername: dockerhost / hostname: private ip address / username: dockeradmin를 입력하고 비밀번호는 advanced를 클릭해 입력한다. password는 아까 지정한 dockeradmin비밀번호인데 대신 path to key를 사용하고 싶다면 dockeradmin으로 접속해 `ssh-keygen`사용해 생성하고 이 키가 위치한 주소를 입력
  - Add Dockerhost to Jenkins `configure systems`
- new Jenkins job to pull the code from GitHub. 메이븐으로 빌드해서 도커 호스트에 복사하기
  - 이번엔 jenkins job을 만들 때 기존에 있던 BuildAndDeployJob을 복사해서 수정한다.
  - 원래 configure에서 post-build actions에서 war파일을 tomcat으로 deploy했으나 이걸 삭제하고 `send build artifacts over ssh`를 택해서 새로 작성한다.
  - ssh server가 dockerhost뿐이기 때문에 그걸 택한다. source fuiles에서 서버에 업로드할 파일을 택하는데 buildAndDeployJob의 webapp/target에 webapp.war존재한다. war파일이 여러개일 수 있어 *.war를 사용한다. 
  - 그리고 remove prefix에서 앞의 webapp/target은 지워준다.
  - 빌드가 성공하면 home/dockeradmin에 webapp.war이 보인다.
- 빌드파일 도커 이미지로 만들기
  - cd /opt (유닉스 계열 운영체제는 /opt에 응용프로그램들이 설치) -> mkdir docker -> 이 파일은 docker admin에 의해 복사된 파일들이 위치하므로 owner를 docker admin으로 변경한다.
  - ` chown -R dockeradmin:dockeradmin /opt/docker`
  - `cd /root`의 Dockerfile을 ` mv Dockerfile /opt/docker/`로 이동하고 파일 owner도 변경한다.
  - post-build actions의 remote directory에 복사될 폴더 지정은 `//opt//docker`인데 /opt/docker일 경우 dockeradmin의 home 위치에서 생성되므로 더블 슬래시를 쓴다.
  - Dockerfile에서도 COPY를 사용해 war파일을 복사하는데 `COPY ./*.war /usr/local/tomcat/webapps`로 앞서 작성했던 catalina_home의 위치다.
  - build로 이미지를 빌드하고 container를 run한다.
- 코드 깃허브 커밋 -> 빌드해서 결과물 도커 호스트에 복사 -> 이미지 생성 -> 컨테이너 생성을 자동으로
  - Send build artifacts over SSH에서 Exec command에 opt/docker로 이동해 build, run하는 절차를 입력한다. 반드시 `;`을 줄마다 입력해야 한다.
  - 만들어진 이미지와 컨테이너를 전부 중단-삭제한 후 빌드하면 살펴보면 자동으로 이미지, 컨테이너가 생성된것이 보인다.
  - 그런데 내용 수정 후 github에서 푸시하고 보면 conflict발생. 컨테이너 네임이 앞선 빌드와 동일하기 때문이다.
  - run절차에 docker stop과 docker rm으르 추가해 앞서만들어졌던 같은 이름의 컨테이너를 삭제하는 절차를 추가하면 정상적으로 작동한다. 하지만 이런 절차를 더 쉽게하도록 `Ansible`을 사용하도록 한다.



## 5. Integrating Ansible in CI/CD pipeline

- deployment tool로 사용해 jenkins가 관리 일을 하지 않게 한다.
- jenkins가 깃허브에서 코드를 가져와서 빌드하고 결과물을 ansible 서버에 복사하면 ansible 서버가 이미지를 생성해서 dockerhub에 푸시하고 컨테이너를 배포한다.
- Ansible 서버 준비
  - setup ec2: 존재하는 보안그룹과 키그룹 택해서 생성
  - setup hostname: `vi /etc/hostname`에서 다 삭제하고 ansible server로
  - create ansible admin user: useradd, passwd로 설정
  - add user to sudoers file: `visudo`로 %wheel ALL=(ALL) NOPASSWD:ALL 아래 ansadmin도 ALL내용 추가하면 ansadmin에도 비밀번호 없이 명령을 실행하는 권한을 부여할 수 있다. 
  - enable password based login: `vi /etc/ssh/sshd_config`에서 Password를 찾아 패스워드authentication활성화하고 `service sshd reload`
  - generate ssh keys: 새로운 세션을 열어 ansadmin으로 로그인한 후 `ssh-keygen`으로 생성. 
  - install ansible: `yum install ansible`을 입력하면 `amazon-linux-extras install ansible2`을 입력해야한다고 알려줌.
- dockerhost를 ansible에 추가해 관리하게 하기. playbook이 dockerhost에게 어떻게 컨테이너를 만들어야 하는지 알려준다.
- On Ansible Node
  - Add docker host IP address: ansible server와 docker host 각각 로그인. dockerhost에도 usradd  passwd ansadmin, vi sudo로 권한 추가까지 실행. password authentication은 잘 설정되어있는지 보기 위해 `grep Password /etc/ssh/sshd_config`로 확인하면된다.
    - 그리고 ansible server root로 docker host를 매니저로 추가해야한다. `vi /etc/ansible/hosts`에 원래 있던 내용을 지우고 docker host의 private ip를 추가한다.
  - Copy ssh keys: ansadmin에서 살펴보면 .ssh에 비밀키와 공개키가 있다. 공개키 id를 타겟 시스템(private ip)에 복사해야 한다. `ssh-copy-id privateIp`를 입력. private ip를 사용하는 이유는 모두 같은 VPC에 속해야 하기 때문이다. 그리고 dockerhost에서 ansadmin으로 로그인해 .ssh를 보면 authorized_keys가 존재한다. 내용을 보면 ansible-server의 공개키와 같다.
    - VPC: Virtual Private Cloud, 사용자가 정의한 가상 네트워크. 가상사설망. 실제로 같은 네트워크상에 있을 때 네트워크를 분리하고 싶으면 기존 공사하는 대신 가상 사설망을 사용해 다른 네트워크인 것처럼 동작하게 한다.
  - Test the connection: ansible-server의 ansadmin에서 home으로 이동해 `ansible all -m ping`으로 연결을 확인한다. host file에 하나의 시스템만 존재해 연결되고 성공 메세지가 나온다. 또 `ansible all -m command -a uptime`과 docker host에서 `uptime`을 확인하면 시간이 같다.
- Integrate Ansible with Jenkins
  - 젠킨스가 빌드 결과물을 ansible system에 복사해 이미지를 만들거나 도커 호스트에 컨테이너를 배포할 수 있게 한다.
  - manage jenkins - configure system - publish over ssh에서 ansible-server를 추가. private ip, ansadmin을 추가한다. advanced에서 비밀번호 인증 추가
  - new Item - Copy_Artifacts_onto_Ansible - BuildAndDeployOnContainer복사. pollSCM을 제거하고 post-build actions에서 ansible-server로 변경하고 exec command제거. 복사될 directory는 mobaxterm에서 직접 생성한 후 ownership을 root가 아닌 ansadmin으로 한다. 빌드 후 살펴보면 webapp.war가 복사되어 있다.
  - 이제 absible-server에서 복사된걸 바탕으로 docker image를 만든다. 파일이 있는 /opt/docker에서 도커 설치 - 실행 후 `usermod -aG docker ansadmin`로 docker그룹에 ansadmin을 추가한다.
  - Dockerfile을 도커 서버에 있던 것과 같은 내용으로 만든다.(톰캣 이미지, webapps이동, .war파일 복사등) 그리고 빌드 후 tomcat 컨테이너 run
- Using Ansible to create containers
  - ansible inventory: 여러 호스트 관리. 호스트 목록이나 그룹 지정하는 파일. 기본 `/etc/ansible/hosts`
  - playbook사용할 때 어디서 할지 알려줘야 하기 때문에 -i옵션(인벤토리가 어디 있는지) 사용. 안쓰면 default위치. 따라서 저 파일을 수정해서 이미 있는 dockerhost ip외에 ansible ip주소를 추가하고 []안에 그룹명을 지정한다.
  - ansible 서버로 일하려면 ssh key를 ansible서버에 복사해야하므로 `ssh-copy-id localhost(ansible-server)`
  - 이제 ansiblel playbook 작성. /opt/docker에서 regapp.yml이라는 파일 생성하고 `ansible-playbook regapp.yml --check`로 실행
  - `docker login`으로 도커 허브 로그인 후 regapp 이미지 푸시

```yaml
---
- hosts: ansible  # 어떤 시스템에서 실행할지

  tasks:  # 할 일
  - name: create docker image
    command: docker build -t regapp:latest .
    args:
      chdir: /opt/docker  # 파일 다른 곳으로 이동해도 commad 실행 위치
```

- Jenkins job to build an image onto ansible
  - 앞서 작성한 regapp.yml에서 task를 추가한다. 생성한 이미지를 dockerhub에 올리기 위한 이름으로 변경해 복사하고 push하는 2개의 절차를 추가한다.
  - 그리고 jenkins에서 post-build actions의 exec command에 ansible playbook실행하는 코드를 추가(regapp.yml 위치 정확히)하면 github에서 변경된 소스코드에 따라 도커 이미지가 잘만들어져서 성공한다.
  
- How to create container on dockerhost using ansible playbook - DevOps Project

  - ansible 서버에서 새로운 yml 파일을 작성한다. 도커 허브의 이미지를 사용해 도커 호스트에서 새로운 컨테이너를 실행하는 명령이다. ansible-playbook deploy_regapp.yml로 실행하면 권한 에러가 발생하는데 도커 서버에서 chmod 777 /var/run/docker.sock으로 권한을 부여하고 다시 실행한다. 하지만 앞선 방법과 마찬가지로 2번 실행하면 같은 이름의 컨테이너가 존재해 에러가 발생한다.

  ```yaml
  - hosts: dockerhost
  
    tasks:
    - name: create container
      command: docker run -d --name regapp-server -p 8082:8080 asj12896/regapp:latest
  ```

- Continuous deployment of docker container using ansible playbook
  - remove existing container / remove existing image / create new container
  - yml파일에 현재 동작중인 컨테이너를 멈추고, 삭제하고 이미지까지 삭제한 후 다시 만드는 절차를 추가한다. 맨 처음에 시작할 땐 삭제해야할 것이 없으므로 ignore_erros: yes까지 각 절차에 추가한다.
- Jenkins CI/CD to deploy on container using Ansible
  - 자동 배포를 위해  jenkins에서 빌드 후 조치 명령에 sleep 10; ansible-playbook /opt/docker/deploy_regapp.yml;을 추가한다.



##  6. Kubernetes on AWS

- 갑자기 도커 컨테이너가 다운되면 복구할 방법이 없는데 이 때 도커 네이티브 서비스인 도커 스웜을 사용한다. 그리고 컨테이너 관리 서비스인 쿠버네티스를 사용하면 더 편리하게 사용할 수 있다. 그래서 도커 컨테이너로 애플리케이션을 배포하지 않고 쿠버네티스 환경으로 배포한다. 

- 아마존 쿠버네티스 서비스인 EKS를 활용하고 이를 쉽게 구성하도록 하는 eksctl활용. 마스터 노드를 구성하지 않아도 aws에서 관리해줘 쉽고 빠르게 이용. 

- 새로운  EKS_Bootstrap_server라는 ec2인스턴스를 생성한다.

- eksctl을 위해 AWS CLI를 최신버전으로 설치한다. eks서버에 기존 1버전대 aws cli가 아닌 awcli2를 설치한다. exit후 다시  root로 접속해 `aws --version`을 보면 변경되어있다. 

- kubectl을 먼저 설치한다. 설치 후 `chmod +x kubectl`을 사용해 kubectl을 사용할 수 있는 권한을 주고 /usr/local/bin으로 이동한다. 

- eksctl을 설치한 후 /tmp에 존재하는 eksctl을 /usr/local/bin으로 이동한다.

  - /usr/local/bin으로 이동해야 외부에서 설치했거나 직접 만든 설치한 명령어를 사용할 수 있다.

- AWS IAM(리소스에 대한 액세스를 안전하게 제어)이 필요하다. 

  - permissions policies에 `ec2FullAccess`,  `cloudFomationFullAccess`, `IAMFullAccess`, `AdministratorAccess`추가
  - role_name: eksctl_role
  - eks server ec2인스턴스에서 보안에 Modify IAM role을 선택해 변경한다.

- Setup Kubernetes using eksctl

  - 클러스터 만들기: nodes-min, max(기본 2씩)나 zone도 설정 가능. node type을 정하지 않으면 큰 인스턴스가 생성되므로 지정해주었다.

  ```
  eksctl create cluster --name {clusterName}  \
  --region {region} \
  --node-type t2.small \
  ```

  - 생성 후 만들어지길 기다리는 동안 15~25분이 걸린다. AWS CloudFormation으로 이동하면 stack에 만들어지고 있다는게 보인다. 다 만들어지면 클러스터와 노드그룹 2개가 보인다. ec2에서도 2가지 인스턴스가 새로 보인다.
  - `kubectl get nodes`에 노드가 2개 있다. 
  - `kubectl run webapp --image=httpd`로 webapp이라는 pod를 생성하고 `kubectl get po`로 확인. 
    - httpd: 아파치 하이퍼텍스트 전송 프로토콜 서버

- Setup Kubernetes using eksctl

  - nginx pod 배포 `kubectl create deployment demo-nginx --image=nginx --replicas=2 --port=80`
  - deployment를 만들면 레플리카셋이 시작된다. 그리고 레플리카셋이 pod를 시작한다. 그리고 expose 명령을 사용해 서비스를 만들고 접근할 수 있다. 여기서 서비스 타입은  loadBalancer
    - 레플리카셋: 정해진 수의 동일한 pod가 항상 실행되도록 한다. 
  - `kubectl get deploy(ment)`, `kubectl get replicaset`, `kubectl get pod`, `kubectl get all`
  - `kubectl expose deployment demo-nginx --port=80 --type=LoadBalancer` / ec2 로드밸런서를 살펴보면 있는 dns name으로 접속가능하고 인스턴스를 살펴보면 2개가 존재한다.
  - 현실에서 이렇게 수동으로 안하고 manifest파일 작성해서 해야함.

- Create 1st manifest file