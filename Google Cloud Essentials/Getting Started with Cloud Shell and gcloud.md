# Getting Started with Cloud Shell and gcloud

## 작업 1: 환경 구성하기

### 기본 리전 및 영역 식별하기
1. 프로젝트 ID를 클립보드에 복사한다.
2. Cloud Shell에서는 다음 gcloud 명령어를 실행하여 <your_project_ID>를 복사한 프로젝트 ID로 바꾼다.
```
gcloud compute project-info describe --project <your_project_ID>
```

출력에서 기본 영역 및 리적 메타데이터 값을 찾는다. 
> `google-compute-default-region` 및 `google-compute-default-zone` 키와 값이 출력에서 빠져 있는 경우 기본 영역이나 리전이 설정되지 않은 것입니다.

### 환경 변수 설정
test