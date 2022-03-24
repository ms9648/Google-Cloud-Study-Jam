# Set Up Network and HTTP Load Balancers
네트워크 부하 분산기와 HTTP 부하 분산기의 차이점과 Compute Engine VM에서 실행되는 애플리케이션용 부하 분산기를 설정하는 방법을 알아보자.

### Load banlancing?
Load balancing refers to efficiently distributing incoming network traffic across a group backend servers, also known as a server farm or server pool.

즉, 네트워크 트래픽은 백엔드 서버에 효율적으로 분산처리하게 하여 서버의 로드율을 증가시키는 것이다.

## 실습 내용
- 네트워크 부하 분산기 설정
- HTTP 부하 분산기 설정
- 네트워크 부하 분산기와 HTTP 부하 분산기의 차이점

## 작업 1: 모든 리소스에 대한 기본 리전 및 영역 설정
1. Cloud shell에서 기본 영역 설정
```
gcloud config set compute/zone us-central1-a
```

2. 기본 리전 설정
```
gcloud config set compute/region us-central1
```

## 작업 2: 다중 웹 서버 인스턴스 만들기
Compute Engine VM 인스턴스 3개를 만들고 거기에 Apache를 설치해보자. 그런 다음 HTTP 트래픽이 인스턴스에 도달할 수 있도록 방화벽 규칙을 추가해보자.

1. 기본 영역에 새 가상 머신 3개를 만들고 모두 같은 태그를 지정한다. 태그 필드를 설정하면 방화벽 규칙을 사용할 때처럼 이러한 인스턴스를 한 번에 참조할 수 있다. 또, 이 명령어는 각 인스턴스에 Apache를 설치하고 고유한 홈페이지를 제공한다.

> Apache: www서버용 소프트웨어로서 HTTP 아파치 서버로 불린다.

```
gcloud compute instances create www1 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
```

```
gcloud compute instances create www2 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www2</h1></body></html>' | tee /var/www/html/index.html"
```
```
gcloud compute instances create www3 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www3</h1></body></html>' | tee /var/www/html/index.html"
```

2. VM 인스턴스에 외부 트래픽을 허용하는 방화벽 규칙을 만든다.
```
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

인스턴스의 외부 IP 주소를 가져와 주소가 정상적으로 작동되는지 확인해보자.

3. 인스턴스 나열하기
```
gcloud compute instances list
```

4. curl을 사용하여 각 인스턴스가 실행 중인지 확인하자.
```
curl http://[IP_ADDRESS]
```

## 작업 3: 부하 분산 서비스 구성
부하 분산 서비스를 구성하면 가상 머신 인스턴스는 사용자가 구성한 고정 외부 IP 주소로 전송되는 패킷을 수신한다. Compute Engine 이미지로 만든 인스턴스는 이 IP 주소를 처리하도록 자동으로 구성된다.

1. 부하 분산기의 고정 외부 IP주소를 만들자.
```
gcloud compute addresses create network-lb-ip-1 \
 --region us-central1
```

2. 기존 HTTP 상태 확인 리소스를 추가한다.
```
gcloud compute http-health-checks create basic-check
```

3. 인스턴스와 같은 리전에 대상 풀을 추가한다. 서비스가 작동하려면 상태 확인이 필요하기 때문에 상태 확인 명령어를 추가해준다.
```
gcloud compute target-pools create www-pool \
    --region us-central1 --http-health-check basic-check
```

4. 풀에 인스턴스를 추가한다.
```
gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
```

5. 전달 규칙을 추가한다.
```
gcloud compute forwarding-rules create www-rule \
    --region us-central1 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
```

## 작업 4: 인스턴스로 트래픽 전송
부하 분산 서비스를 구성했으므로 이제 전달 규칙에 트래픽을 보내고 트래픽이 여러 인스턴스에 분산되는 것을 확인할 수 있다.

1. 부하 분산기에서 사용하는 www-rule 전달 규칙의 외부 IP 주소 확인하기
```
gcloud compute forwarding-rules describe www-rule --region us-central1
```

2. curl 명령어를 사용하여 외부 IP 주소에 엑세스한다.
```
while true; do curl -m1 IP_ADDRESS; done
```

curl 명령어가 실행되면 인스턴스 세 개에서 무작위로 응답한다. 구성이 완전히 로드되어 인스턴스가 정상으로 표시될 때까지 기다린 후 시도해본다.

Ctrl+C를 눌러 명령어 실행을 중지시킬 수 있다.

## 작업 5: HTTP 부하 분산기 만들기
HTTP 부하 분산은 GFE(Google Front-End)에서 구현된다. URL이 각기 적절한 인스턴스 집합으로 라우팅되도록 URL 규칙을 구성할 수 있다. 요청은 항상 사용자와 가장 가까운 인스턴스 그룹으로 라우팅된다. 가장 가까운 그룹에 용량이 충분하지 않으면 용량이 있는 가장 가까운 그룹으로 요청이 전송된다.

Compute Engine 백엔드로 부하 분산기를 설정하려면 VM이 인스턴스 그룹에 있어야한다. 관리형 인스턴스 그룹은 외부 HTTP 부하 분산기의 백엔드 서버를 실행하는 VM을 제공한다.

1. 부하 분산기 템플릿 만들기
```
gcloud compute instance-templates create lb-backend-template \
   --region=us-central1 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --image-family=debian-9 \
   --image-project=debian-cloud \
   --metadata=startup-script='#! /bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
```

2. 템플릿을 기반으로 관리형 인스턴스 그룹 만들기
```
gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=us-central1-a
```

3. fw-allow-health-check 방화벽 규칙을 만든다. allow-health-check 대상 태그를 사용하여 VM을 식별할 수 있다.
```
gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80
```

4. 인스턴스가 실행 중이므로 고객이 부하 분산기에 연결하는 데 사용하는 전역 고정 외부 IP 주소를 설정한다.
```
gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global
```

예약된 IPv4 주소를 확인한다.
```
gcloud compute addresses describe lb-ipv4-1 \
    --format="get(address)" \
    --global
```

5. 부하 분산기용 상태 확인을 만든다.
```
    gcloud compute health-checks create http http-basic-check \
        --port 80
```

6. 백엔드 서비스를 만든다.
```
    gcloud compute backend-services create web-backend-service \
        --protocol=HTTP \
        --port-name=http \
        --health-checks=http-basic-check \
        --global
```

7. 백엔드 서비스에 인스턴스 그룹을 백엔드로 추가한다.
```
    gcloud compute backend-services add-backend web-backend-service \
        --instance-group=lb-backend-group \
        --instance-group-zone=us-central1-a \
        --global
```

8. URL 맵을 만들어 들어오는 요청을 기본 백엔드 서비스로 라우팅한다.
```
    gcloud compute url-maps create web-map-http \
        --default-service web-backend-service
```

9. 대상 HTTP 프록시를 만들어 URL 맵에 요청을 라우팅한다.
```
    gcloud compute target-http-proxies create http-lb-proxy \
        --url-map web-map-http
```

10. 들어오는 요청을 프록시로 라우팅하는 전역 전달 규칙을 만든다.
```
    gcloud compute forwarding-rules create http-content-rule \
        --address=lb-ipv4-1\
        --global \
        --target-http-proxy=http-lb-proxy \
        --ports=80
```

## 작업 6: 인스턴스로 전송되는 트래픽 테스트
1. Cloud Console의 탐색 메뉴에서 네트워크 서비스 > 부하 분산으로 이동한다.

2. 방금 만든 부하 분산기(web-map-http)를 클릭한다.

3. 백엔드 섹션에서 백엔드 이름을 클릭하여 VM이 정상 상태인지 확인한다. VM이 정상 상태가 아니라면 잠시 기다렸다가 페이지를 새로고침 해본다.

4. VM이 정상 상태이면 웹브라우저에서 http://IP_ADDRESS/ 로 이동하여 부하 분산기를 테스트한다. 여기서 IP_ADDRESS를 부하 분산기의 IP 주소로 바꾼다.

브라우저는 페이지를 제공한 인스턴스의 이름과 영역을 표시하는 콘텐츠로 페이지를 렌더링해야 한다.