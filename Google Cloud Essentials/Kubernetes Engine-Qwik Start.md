# Kubernetes Engine: Qwik Start
GKE(Google Kubernetes Engine)에서는 Google 인프라를 사용하여 컨테이너식 애플리케이션을 배포, 관리 및 확장할 수 있는 관리형 환경을 제공한다. 즉, 리눅스 컨테이너 작업을 자동화하는 플랫폼이다. Kubernetes Engine 환경은 컨테이너 클러스터를 형성하도록 그룹화된 여러 머신(Compute Engine instance)으로 구성되어 있다. 이번 실습에서는 GKE를 사용하여 직접 컨테이너를 생성하고 애플리케이션을 배포해 볼 것이다.

## 컨테이너 개념 잡기
### 컨테이너란?
컨테이너는 앱 실행에 필요한 바이너리, 라이브러리, 구성 파일 등 모든 코드와 종속성을 포함하는 표준화된 소프트웨어 유닛이다. 컨테이너화된 소프트웨어는 한 컴퓨팅 환경에서 다른 컴퓨팅 환경으로 이동하여 안정적으로 실행될 수 있다.

### 컨테이너 VS 가상머신
가상머신과 컨테이너 모두 가상화 기술을 통해 애플리케이션 실행을 위한 독립된 환경을 구축한다. 가장 큰 차이점은 두 기술이 가상화하는 세분화된 영역에 있다. 가상머신은 운영 체제/머신 수준의 가상화를 제공하는 반면에 컨테이너는 소프트웨어 수준의 가상화를 제공한다.

## Google Kubernetes Engine을 사용한 클러스터 조정
GKE 클러스터는 Kubernetes 오픈소스 클러스터 관리 시스템을 기반으로 한다.

> 클러스터: 여러 대의 컴퓨터들이 연결되어 하나의 시스템처럼 동작하는 컴퓨터들의 집합

Kubernetes는 컨테이너 클러스터와 상호작용할 수 있는 메커니즘을 제공한다. Kubernetes 명령어와 리소스를 사용하면 애플리케이션을 배포 및 관리하고 관리 작업을 수행하고 정책을 설정하며 배포된 워크로드의 상태를 모니터링할 수 있다.

> 워크로드: 쿠버네티스에서 구동되는 애플리케이션

Kubernetes는 널리 쓰이는 Google 서비스와 동일한 설계 원칙을 따르고 있어 자동 관리, 애플리케이션 컨테이너의 모니터링 및 활성 여부 조사, 자동 확장, 순차적 업데이트와 같은 이점을 그대로 누릴 수 있다. 

## Google Cloud에서 사용하는 Kubernetes
GKE 클러스터를 실행하면 Google Cloud의 고급 클러스터 관리 역량을 활용할 수 있다는 장점이 있다.
- Compute Engine 인스턴스를 위한 부하 분산
    - Google Cloud는 서버 측 부하 분산 기능을 제공하기 때문에 들어오는 트래픽을 여러 가상 머신 인스턴스에 분산할 수 있다.
- 노드 풀로 클러스터 안에 하위 노드 집합을 지정하여 유연성 강화
- 클러스터에서 노드 인스턴스 개수 자동 확장
- 클러스터에서 노드 소프트웨어 자동 업그레이드
- 노드 자동 복구로 노드 상태 및 가용성을 유지 관리
- Cloud Monitoring을 통한 로깅 및 모니터링으로 클러스터 현황에 대한 가시성 확보

## 작업 1: 기본 컴퓨팅 영역 설정
컴퓨팅 영역이랑 리전 내에 대략적으로 클러스터와 리소스가 존재하는 위치를 의미한다. 예를 들어 us-central1-a는 us-central1 리전에 속한 영역이다.

1. 컴퓨팅 영역의 기본값을 us-central1-a로 설정하려면 Cloud Shell에서 새 세션을 시작하고 다음 명령어를 실행한다.
```
gcloud config set compute/zone us-central1-a
```
출력:
```
Updated property [컴퓨팅/영역].
```

## 작업 2: GKE 클러스터 만들기
클러스터는 1개 이상의 클러스터 마스터 머신과 노드라는 다수의 작업자 머신으로 구성된다. 노드란 클러스터를 구성하기 위해 필요한 Kubernetes 프로세스를 실행하는 Compute Engine VM 인스턴스이다.

1. 클러스터를 생성하기
원하는 이름으로 클러스터 이름을 정하자.
> 클러스터 이름은 문자로 시작하고 영숫자로 끝나야 하며 40자를 초과할 수 없다.
```
gcloud container clusters create [CLUSTER-NAME]
```

## 작업 3: 클러스터의 사용자 인증 정보 얻기
클러스터를 만든 후 클러스터와 상호작용하려면 사용자 인증 정보가 필요하다.
1. 클러스터 인증하기
```
gcloud container clusters get-credentials [CLUSTER-NAME]
```

## 작업 4: 클러스터에 애플리케이션 배포
이제 클러스터에 컨테이너식 애플리케이션을 배포할 수 있다. hello-app을 클러스터에서 실행해보자

GKE는 Kubernetes 객체를 사용하여 클러스터의 리소스를 만들고 관리한다. 웹 서버와 같은 stateless 애플리케이션을 배포할 때는 Kubernetes에서 배포 객체를 사용한다. 서비스 객체는 인터넷에서 애플리케이션에 액세스하기 위한 규칙과 부하 분산 방식을 정의한다.

1. hello-app 컨테이너 이미지에서 새 배포 hello-server 생성하기
```
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```
이 Kubernetes 명령어를 사용하면 hello-server를 나타내는 배포 객체가 생성된다. 여기서 --image는 배포할 컨테이너 이미지를 지정한다. 해당 명령어는 Container Registry 버킷에서 예시 이미지를 가져온다. gcr.io/google-samples/hello-app:1.0은 가져올 특정 이미지 버전을 나타낸다. 버전이 지정되지 않은 경우 최신 버전이 사용된다.

2. 애플리케이션을 외부 트래픽에 노출할 수 있는 Kubernetes 리소스인 Kubernetes Service를 생성하려면 다음 kubectl expose 명령어를 실행한다.
```
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```
- '--port'를 통해 컨테이너가 노출될 포트가 지정된다.
- 'type="LoadBalancer"'는 컨테이너의 Compute Engine 부하 분산기를 생성한다.

3. hello-server 서비스를 검사하려면 kubectl get을 실행한다.
```
kubectl get service
```

> EXTERNAL-IP 열이 대기중 상태이면 위 명령어를 다시 실행하자.

4. 웹브라우저에서 애플리케이션을 보려면 새 탭을 열고 다음 주소를 입력한다. 
```
http://[EXTERNAL-IP]:8000
```

## 작업 5: 클러스터 삭제
1. 클러스터를 삭제하려면 다음 명령어를 실행한다.
```
gcloud container clusters delete [CLUSTER-NAME]
```

2. 메시지가 표시되면 Y를 입력하여 확인한다.
