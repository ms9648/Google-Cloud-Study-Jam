# Compute Engine: Qwik Start - Windows

Compute Engine에서 Windows Server 인스턴스를 실행하고 원격 데스크탑 프로토콜(RDP)을 사용하여 연결하는 방법을 알 수 있다.

## 가상 머신 인스턴스 만들기

1. Cloud Console의 탐색 메뉴에서 Compute Engine > VM 인스턴스를 클릭하고 인스턴스 만들기를 클릭한다.

2. 머신 구성 섹션의 시리즈에서 N1을 선택한다.

3. 부팅 디스크 섹션에서 변경을 클릭하여 부팅 디스크 구성을 시작한다.

4. 운영체제에서 Windows Server를 선택하고 버전에서 Windows Server 2012 R2 Datacenter를 선택한 후 클릭한다. 나머지 설정은 기본값으로 둔다.

5. 인스턴스를 만든다.

## 원격 데스트탑(RDP)을 통해 Windows Server에 접속하기

### Windows 시작 상태 테스트하기
잠시 후 Windows Server 인스턴스가 프로비저닝되며 초록색의 상태 아이콘과 함께 VM 인스턴스 페이지에 표시된다.

단, 모든 OS 구성요소를 초기화하는 데 다소 시간이 걸리므로 아직 서버 인스턴스에 RDP 연결을 수락할 준비가 되어 있지 않을 수 있다.

서버 인스턴스에서 RDP 연결을 수락할 준비가 되었는지 확인하려면 Cloud Shell 터미널 명령줄에서 다음 명령어를 실행한다.
```
gcloud compute instances get-serial-port-output instance-1
```

메세지가 표시되면 n을 입력하고 Enter 키를 누른다.

다음 명령어 결과가 나올 때까지 명령어 입력을 반복한다. 이 결과는 OS 구성요소가 초기화되었으며 Windows Server가 RDP 연결을 수락할 준비가 되었다는 의미이다.
```
------------------------------------------------------------
Instance setup finished. instance-1 is ready to use.
------------------------------------------------------------
```
#### RDP를 통해 Windows Server에 접속하기
RDP 로그인 비밀번호를 설정하려면 Cloud Shell 터미널에서 다음 명령어를 실행한 다음 [instance]를 사용자가 만든 VM 인스턴스로 교체하고 [username]을 'admin' 으로 설정한다.
```
gcloud compute reset-windows-password [instance] --zone us-central1-a --user [username]
```

```
If asked Would you like to set or reset the password for [admin] (Y/n)? 
```
라는 문구가 뜨면 Y를 입력한다.

Windows 사용 여부에 따라 여러 가지 방법으로 RDP를 통해 서버에 연결할 수 있다.

Google Cloud 이벤트에서 Chromebook 또는 기타 컴퓨터를 사용하는 경우 이미 컴퓨터에 RDP 앱이 설치되어 있을 가능성이 높다. 화면 왼쪽 하단에 아이콘이 표시된다면 아이콘을 클릭하고 VM의 외부 IP를 입력한다.

Windows를 사용하지 않지만 Chrome을 사용 중이라면 Chrome RDP for Google Cloud Platform 확장 프로그램을 사용하여 브라우저에서 직접 RDP를 통해 서버에 연결할 수 있다.(RDP 클릭)

이렇게 하면 Chrome RDP 확장 프로그램을 설치하라는 메시지가 표시된다. 확장 프로그램이 설치되면 로그인 페이지가 표시되며 여기에서 앞서 언급된 명령어의 결과를 기반으로 Windows 사용자 이름을 관리자로 지정하고 비밀번호를 설정하여 로그인할 수 있다.
> Domain: field는 무시해도 된다.

또한 Windows 컴퓨터를 사용 중이라면 RDP 메뉴에서 선택하여 RDP 파일을 다운로드할 수 있다.

Windows에서는 RDP 파일을 더블클릭하고 Windows 사용자 및 비밀번호를 사용해 로그인하면 된다.

Macintosh를 사용하는 경우 CoRD와 같이 무료로 엑세르하여 설치할 수 있는 RDP 클라이언트 패키지가 여러 개 있다. 설치 후 위와 같이 Windows 서버의 외부 IP 주소로 연결한다. 연결된 후 로그인 페이지가 표시되며 여기에서 앞서 언급된 명령어의 결과를 기반으로 Windows 사용자 이름을 'admin'으로 지정하고 비밀번호를 설정하여 로그인할 수 있다.

로그인이 완료되면 Windows 데스크탑이 표시된다.

