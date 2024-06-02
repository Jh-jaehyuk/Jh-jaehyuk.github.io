---
title: Infrastructure Docker - 01
date: 2024-06-02 12:41:45 +09:00
categories: [Infrastructure, Docker]
tags: 
    [
        Infrastructure,
        Docker
    ]
---

# Instrastructure
![Infra main](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/44db627f-cf0a-43c3-a922-3292373b7914){: .align-center}  
이번 글에서는 저번 회고록에서 말한 것과 같이 인프라에 대해 공부한 내용에 대해 작성하고자 합니다. 우선 인프라 관련해선 처음 작성하는 글이기 때문에, 간단히 인프라에 대해 알아보도록 하겠습니다.  
IT 분야에서의 인프라(Infrastructure)는 정보기술 시스템과 서비스를 구축하고 운영하기 위해 필요한 모든 물리적, 비물리적(가상) 자원의 집합을 의미합니다. 다소 말이 어렵게 느껴지지만 거기에 포함되는 요소들을 듣는다면 대충 어떤 느낌인지 감이 잡힐 것이라고 생각합니다.  

### 주요 구성 요소

1. 하드웨어
 * 서버: 데이터 처리와 애플리케이션 호스팅을 위한 컴퓨터 시스템
 * 스토리지: 데이터 저장을 위한 장치 (HDD, SSD 등)
 * 네트워크 장비: 라우터, 스위치, 방화벽 등
  
2. 소프트웨어
 * 운영 체제(OS): 서버와 클라이언트 시스템에서 실행되는 기본 소프트웨어 (Linux, Windows, Mac 등)
 * 미들웨어: 다양한 애플리케이션과 데이터 간의 통신을 지원하는 소프트웨어 (데이터베이스 관리 시스템)
  
3. 네트워크
 * LAN / WAN: 로컬 및 광역 네트워크 인프라
 * 인터넷 연결: 외부 네트워크와의 통신을 위한 연결
  
4. 데이터 스토리지
 * 데이터베이스: 정형 및 비정형 데이터를 저장하고 관리하는 시스템
 * 백업 및 복구 시스템: 데이터 손실에 대비하는 시스템
  
5. 가상화 및 컨테이너화
 * 가상 머신(VM): 물리적 하드웨어 위에서 실행되는 소프트웨어 기반의 컴퓨터
 * 컨테이너: Docker와 같은 기술을 사용하여 애플리케이션과 그 종속성을 패키징하여 일관된 실행 환경을 제공
 * 오케스트레이션: Kubernetes와 같은 도구를 사용하여 컨테이너의 배포, 확장, 관리를 자동화
  
6. 클라우드 서비스
 * IaaS (Infrastructure as a Service): AWS, Azure, GCP 등에서 제공하는 가상화된 컴퓨팅 자원
 * PaaS (Platform as a Service): 개발 및 배포를 위한 플랫폼 서비스
 * SaaS (Software as a Service): 사용자가 인터넷을 통해 접근하는 소프트웨어 애플리케이션
  
위에 설명드린 것과 같은 IT 인프라는 어떤 하나의 요소와 기술만을 포함하는 것이 아니라, 다양한 구성 요소와 기술로 이루어져 있음을 알 수 있습니다. 그렇기 때문에 이들 간의 상호작용을 통해 안정적이고 효율적인 시스템 운영이 가능하도록 하는 것이 핵심이라고 할 수 있습니다.  

## Docker - 01
저는 인프라의 여러 요소들 중에서 **Docker**와 **Kubernetes**에 대해 먼저 공부할 예정입니다. 그 중에서도 가장 먼저 시작할 것은 **Docker**입니다.  
Docker는 애플리케이션과 그 종속성을 함께 패키징하여 **컨테이너**라는 독립된 환경에서 실행할 수 있게 해주는 플랫폼입니다. 컨테이너는 가상머신보다 더 가볍고 빠르며, 일관된 환경을 제공하여 개발부터 배포까지의 과정을 단순화한다는 장점이 존재합니다. 이런 장점들덕분에 Docker 다루는 법을 알아야 실무에서도 유리하다고 생각하여 Docker를 먼저 공부하기로 했습니다!  

### Installation
Docker를 사용하기 전, 제가 설치한 것은 *Docker Desktop*, *Kubectl*, *Minikube* 세 가지입니다. 설치 과정을 하나씩 살펴봅시다.  
모든 과정은 **Mac Sonoma**기준으로 진행하였습니다.  

#### Docker Desktop
1. [Docker Desktop 공식 페이지](https://www.docker.com/products/docker-desktop/)에 접속하여 본인 OS에 맞는 설치 프로그램 다운로드
2. 설치 완료된 Docker Desktop을 실행
3. 상단 좌측 메뉴의 "Extensions"에서 **Setting**을 선택
![docker desktop setting1](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/32e005cd-7681-4653-9bfc-a919813412d7){: .align-center}  
4. "Setting"의 왼쪽 메뉴에서 **Kubernetes**를 선택한 후, **Enable Kubernetes** 옵션을 체크
![docker desktop setting2](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/5493a8d8-f512-4157-b53d-a273ffc63eff){: .align-center}  
  
#### Kuberctl
1. Homebrew를 사용하여 **kuberctl**을 설치

```bash
brew install kubectl
```
  
**kubectl**은 Kubernetes API와 상호작용하여 클러스터의 자원을 조작하는 명령어 도구입니다.
  
#### Minikube
1. Homebrew를 사용하여 **minikube**를 설치

```bash
brew install minikube
```
  
2. Minikube를 시작하고 Kubernetes 클러스터 구축

```bash
minikube start
```
  
Minikube를 사용하면 로컬 환경에서 Kubernetes 클러스터를 쉽게 만들고 관리할 수 있습니다.
  
### TASK
간단한 애플리케이션을 작성해보고 Docker image를 build하여 레포지토리에 push해보도록 하겠습니다.  

#### 애플리케이션 작성
주소로 get 요청하면 "hello k8s!" 응답을 하는 애플리케이션을 아래와 같이 작성하였습니다.  

```javascript
//app.js

const express = require("express");

const app = express();

app.get("/", (req, res) => {
    res.send("hello k8s!");
});

app.listen(3000, () => console.log("server start"));
```
  
자바스크립트로 코드를 짜본 경험이 없어서, 우선은 샘플 코드를 참고하여 작성해보았습니다.  

#### Build Docker image
그 다음으로는 Docker image를 빌드하기 위한 *Dockerfile*을 아래와 같이 생성하였습니다.  

```dockerfile
FROM node:16-alpine

WORKDIR /my/work/dir

COPY package*.json ./

RUN npm install

EXPOSE 3000

CMD ["node", "app.js"]
```
  
터미널에서 Docker build 명령어를 사용하여 빌드를 진행하였습니다.
![docker build fail1](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/47aef102-0a87-452c-b8ef-e3947429b402){: .align-center}
예상과는 다르게 오류가 발생했습니다! 오류에 대한 내용을 검색해보니 **node.js**가 설치되지않아서 발생하는 문제인 것 같았습니다. 그래서 아래와 같이 Homebrew를 이용하여 node.js 설치를 진행하였습니다.
![install nodejs1](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/6ebdb217-788d-4012-b452-c7bcde2290e4){: .align-center}  
그리고 설치가 제대로 됐는지 확인해보았습니다.  
![install nodejs](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/16479942-6224-46c1-ba14-83ea17e9fe60){: .align-center}  
오류없이 버전이 출력되는 것을 보아하니, 제대로 설치가 된 것 같습니다! 이제 빌드를 다시 시도해봅시다!  
![docker build1](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/46895b99-c9e7-452d-b89d-646db949cf3c){: . align-center}  
제대로 빌드가 된 것을 확인할 수 있습니다.  

#### Push Docker image
Docker image가 제대로 빌드되었으니 Docker hub에 push해보도록 하겠습니다.  
![docker push fail1](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/d929dbd2-d8e7-4fbc-bc20-fa38e8ac4d0e){: .align-center}  
이번에도 예상과 다르게 오류가 발생했습니다. 이 쯤 되니까 슬슬 피곤해졌습니다.:sob:  
관련 오류 메시지를 알아보니 로그인이 제대로 되지 않는 것 같았습니다. 분명 docker login를 해주었는데도 불구하고 실제로는 로그인이 되어있지 않은 것 같았습니다.  

```bash
cd ~/.docker/
```
  
위와 같이 폴더 이동하여, 로그인 정보가 담겨져 있는 내부의 config.json 파일을 삭제하였습니다. 다시 로그인하기 위해 아래와 같이 입력하고 로그인을 진행하였습니다.

```bash
docker login -u j213h --password-stdin
```
  
![docker login success](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/44a8dd2b-3658-4eec-8fc9-5359d6f2fdee){: .align-center}  
성공적으로 로그인되었습니다. 하지만 그 이후로도 push 명령을 실행하면 권한이 없다는 오류가 계속 반복되었습니다. 문제를 해결하기 위해서 계속 구글링을 해본 결과, 제가 처음에 빌드했던 docker image 이름 자체가 잘못되었을수도 있다는 정보를 얻게 되었고 아래가 같이 수정해보았습니다.

```
task01-sample/task01-sample-app => j213h/task01-sample-app
```
  
이렇게 수정하여 빌드하고 push를 해보았습니다.
![docker push success1](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/6e5e9069-330e-4f05-b05e-5d3c40336e37){: .align-center}  
드디어 성공적으로 push 되었습니다!  

#### Docker run
레포지토리에 성공적으로 Push하고 제대로 실행되는지 확인하기 위해서 docker run을 해보았습니다. app.js를 찾을 수 없다는 오류메시지가 나왔습니다. 분명 app.js가 있는 폴더를 WORKDIR로 잘 지정해준 것 같았는데.. 뭔가 잘못된 것 같습니다. 다시 구글링의 시간입니다.  
우선 app.js, dockerfile, package.json 이 3개의 파일을 같은 곳으로 옮겨봅니다. 그래도 해결되지 않습니다! 그렇다면 작성한 파일 내부를 수정해보아야 할 것 같습니다. 

```javascript
// app.js
const express = require("express");
const PORT = 8080;
const app = express();

app.get("/", (req, res) => {
    res.send("hello k8s!");
});

app.listen(PORT, () => console.log("server start"));
```
  
위와 같이 별도 포트 설정을 추가해주었습니다. 그리고 dockerfile은 아래와 같이 수정하였습니다.  

```dockerfile
FROM node:16-alpine

WORKDIR /Users/j213h/Documents/Infra/TASK01/dockerfile-folder

COPY ./ ./

RUN npm install

EXPOSE 3000

CMD ["node", "app.js"]
```
  
package.json과 app.js를 참고할 수 있도록 COPY 부분은 COPY ./ ./으로 수정하였습니다.  
그리고 다시 docker run을 하였는데 이번엔 "express" 모듈이 없다는 메시지가 나왔습니다. 구글링 해보니 간단한 문제여서 바로 모듈을 설치해주었습니다. 그리고 다시 docker run을 해보았습니다.  
![docker run success](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/025704ce-b11e-48ef-89e2-29a7e1b9712e){: .align-center}  
오랜 시간이 걸렸지만 결국 성공했습니다!  
Docker desktop에서도 아래와 같이 확인 가능했습니다.
![docker desktop image](https://github.com/Jh-jaehyuk/Jh-jaehyuk.github.io/assets/126551524/f601688a-7570-45c1-a6ca-546069332d18){: .align-center}  

---  
3일에 거쳐서 조금씩 해결해보았는데 생각보다 쉽지 않았던 것 같습니다. 그리 어려운 걸 시도한 것도 아닌데 이정도로 헤맬 줄 몰랐습니다. 시간이 없어서 다소 급하게 작성한 느낌이 있지만! 다음엔 보다 자세히 기록해보겠습니다.  
이번 결과물 URL : https://hub.docker.com/repository/docker/j213h/task01-sample-app/general  
읽어주셔서 감사합니다!
