# AI Platform: Qwik Start
로컬 및 AI Platform을 기반으로 TensorFlow 2.x 모델 학습을 연습해 볼 것이다. 또한 예측을 위한 모델을 AI Platform에 배포하는 방법을 배워볼 것이다. 미국 census 소득 데이터 셋을 이용하여 한 사람의 소득 범주를 예측하도록 모델을 학습시켜볼 것이다.

## 빌드 대상
소득 범주는 다음과 같다.
- '>50K' : 50,000 달러 초과
- '<=50K' : 50,000 달러 이하

이 샘플은 Keras Sequential API를 사용하여 모델을 정의한다. 또한 인구조사 데이터 셋에 특화된 데이터 변환을 정의하고 잠재적으로 변환된 특성을 모델의 DNN 또는 선형 부분에 할당한다.

## AI Platform Notebooks 실행
1. 탐색 메뉴에서 AI Platform -> Dashboard를 클릭한다.
2. View notebook instances를 클릭한다.
3. 노트북 인스턴스 페이지에서 New Notebook를 클릭한다.
4. 인스턴스 맞춤설정 메뉴에서 TensorFlow Enterprise를 선택하고 TensorFlow Enterprise 2.x(LTS 사용) 최신 버전 > GPU 사용 안 함을 선택한다.
5. 새 노트북 인스턴스 대화상자에서 연필 모양 아이콘을 클릭하여 인스턴스 속성을 편집한다.
6. 인스턴스 이름에는 미리 생성된 기본 이름을 사용하자.
7. 리전에는 us-central1, 영역에는 선택한 리전의 영역을 선택한다.
8. '머신 유형'에서 n1-standard-2를 선택한다.
9. 남은 필드는 그대로 두고 만든다.
10. 몇 분뒤 Jupyter Lab 열기를 클릭하면 JupyterLab 창이 새 탭에서 열린다.

## AI Platform Notebooks 인스턴스 내로 예시 저장소 클론
JupyterLab 인스턴스에서 training-data-analyst 노트북을 클론한다.
1. JupyterLab의 터미널을 연다.
2. 명령 프롬프트에서 다음 명령어를 입력한다.
```
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```

> 구글 클라우드 교육 레포인 듯하다.

3. training-data-analyst 폴더를 클릭하여 폴더 내부 콘텐츠를 볼 수 있는지 확인한다.

### 예시 노트북으로 이동
AI Platform Notebooks에서 training-data-analyst/self-paced-labs/ai-platform-qwikstart로 이동하여 ai_platform_qwik_start.ipynb를 연다.

노트북의 셀을 모두 지우고 셀을 하나씩 실행한다.