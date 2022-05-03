---
title:  "머신러닝 워크플로우(workflow of machine learning) - 2"
excerpt: "머신러닝 일반적인 워크플로우(The universal workflow of machine learning)와 모델 배치(Define the task) 및 마무리 요약"

categories:
 - AI
tags:
 - [machine learning]

toc: true
toc_sticky: true

date: 2021-10-31
last_modified_at: 2021-10-31
---

## **1.3 모델 배치(Deploy your model)**

이제 모델이 테스트 세트에 대한 최종 평가를 성공적으로 마쳤습니다. 이제 배치하고 생산적인 일을 시작할 준비가 되었습니다.

---

### **1.3.1 사용자에게 업무 설명 및 기대치 설정(Explain your work to stakeholders and set expectations)**

성공과 고객 신뢰는 지속적으로 고객의 기대에 부응하거나 그 이상을 달성하는 것입니다. 실제로 제공하는 시스템은 그 그림의 절반에 불과합니다. 나머지 절반은 출시 전 적절한 기대치를 설정하고 있습니다.

AI 시스템에 대한 비전문가들의 기대는 비현실적입니다. 이 문제를 해결하려면 모델 오류의 몇 가지 예를 보여 주는 것처럼 소비자에 대해 고려해야 합니다(잘못 분류된 표본 등).

또한 이전에 사람이 처리하던 과정의 경우 인간 수준의 성능을 기대할 수 있습니다. 대부분의 머신러닝 모델은 인간이 만든 레이블에 근접하도록 (불완전하게) 훈련되었기 때문에 거의 도달하지 못합니다. 모델 성능 기대치를 명확히 전달해야 합니다.

"모델은 98%의 정확도를 가지고 있다“와 같은 추상적인 사용을 피하고, 구체적으로 잘못된 부정 비율과 잘못된 긍정 비율에 대해 설명하세요. "이러한 설정을 사용하면 부정행위 탐지 모델은 5%의 거짓 음성 비율과 2.5%의 거짓 양성률을 갖습니다. 매일 평균 200건의 유효거래가 사기행위로 플래그가 지정돼 수작업 검토를 위해 발송되고, 평균 14건의 사기거래가 누락되었습니다. 평균 266건의 부정거래가 적발될 것입니다." 모델의 성능 측정 기준을 비즈니스 목표와 명확하게 연관시킵니다.

또한 주요 시작 매개 변수(예: 트랜잭션에 플래그를 지정해야 하는 확률 임계값)를 선택할 것인지 이해 관계자와 논의해야 합니다. 이러한 결정에는 비즈니스 맥락을 깊이 이해해야만 처리할 수 있는 협정(tradeoffs)이 포함됩니다.

---

### **1.3.2. 추론 모델 출하(Ship an inference model)**

머신러닝 프로젝트는 훈련된 모델을 저장할 수 있는 코랩 노트북에 도착해도 끝나지 않습니다. 훈련 중에 조작한 것과 동일한 파이썬 모델 객체를 생산에 투입하는 경우는 거의 없습니다. 먼저 파이썬(Python)이 아닌 다른 것으로 모델을 내보내는 것이 좋습니다.

 - 운영 환경에서 Python을 전혀 지원하지 않을 수 있습니다. 예를 들어, Python이 모바일 앱이나 임베디드 시스템인 경우입니다.
 - 나머지 앱이 Python이 아닌 경우(JavaScript, C++ 등) 모델을 서비스하기 위해 Python을 사용하면 상당한 오버헤드(Overhead: 메인 작업에 비해 부수적인 작업의 양이 지나치게 많을 때)가 발생할 수 있습니다.

생산 모델은 훈련용이 아니라 예측(추론 단계) 출력에만 사용되므로 모델을 더 빠르게 만들고 메모리 공간을 줄일 수 있는 다양한 최적화를 수행할 수 있습니다.

사용할 수 있는 다양한 모델 구축 옵션을 간단히 살펴보겠습니다.

---

**REST API 모델 배포**

REST API는 모델을 제품으로 바꾸는 일반적인 방법일 것입니다. 서버나 클라우드 인스턴스에 TensorFlow를 설치하고 REST API를 통해 모델의 예측을 쿼리(정보 요청)합니다.

플라스크(또는 다른 파이썬 웹 개발 라이브러리)와 같은 것을 사용하여 자신만의 서빙 앱을 구축하거나 TensorFlow의 자체 라이브러리를 사용하여 모델을 API로 제공할 수 있습니다. **TensorFlow Serving**(www.tensorflow.org/tfx/guide/serving)을 사용하여 Keras 모델을 몇 분 내에 배포할 수 있습니다.

다음과 같은 경우 이 배포 설정을 사용해야 합니다.

 - 모델의 예측을 소비할 애플리케이션은 인터넷에 신뢰할 수 있는 액세스를 갖게 될 것입니다. 예를 들어 응용 프로그램이 모바일 응용 프로그램인 경우 원격 API에서 예측을 제공한다는 것은 응용 프로그램을 비행기 모드나 저연결 환경에서 사용할 수 없음을 의미합니다.
 - 애플리케이션에는 엄격한 대기 시간 요구사항이 없습니다. 요청, 추론 및 응답 왕복에는 일반적으로 약 500ms가 소요됩니다.
 - 추론을 위해 전송된 입력 데이터는 그다지 민감하지 않습니다. 즉, 데이터는 모델에서 확인할 필요가 있으므로 해독된 형태로 서버에서 사용할 수 있어야 합니다(그러나 HTTP 요청 및 응답에 SSL 암호화를 사용해야 함).

예를 들어 이미지 검색 엔진, 음악 추천 시스템, 신용카드 사기 탐지 프로그램, 위성 이미지 프로젝트는 모두 REST API를 통해 서비스하기에 적합합니다. 유의할 점으로는 REST API로 모델을 배포할 때 중요한 질문은 코드를 직접 호스팅할지 아니면 완전히 관리되는 타사 클라우드 서비스를 사용할지 여부입니다.

---

**장치에 모델 배포**

때로는 스마트폰, 로봇의 내장형 ARM CPU 또는 작은 장치의 마이크로컨트롤러 등 해당 애플리케이션을 실행하는 동일한 장치에 모델을 사용해야 할 수도 있습니다. 다음과 같은 경우 이 설정을 사용해야 합니다.

 - 모델에 엄격한 지연 시간 제약이 있거나 연결성이 낮은 환경에서 실행해야 합니다. 증강 현실 애플리케이션을 구축하는 경우 원격 서버를 쿼리하는 것은 실행 가능한 옵션이 아닙니다.
 - 모델을 대상 장치의 메모리 및 전력 제약 조건에서 실행할 수 있을 정도로 충분히 작게 만들 수 있습니다(TensorFlow Model Optimization Toolkit:을 사용하여 이 문제를 해결할 수 있습니다).
 - 런타임 효율성과 정확도 사이에는 항상 절충이 있기 때문에 메모리 및 전력 제약으로 인해 대형 GPU에서 실행할 수 있는 최상의 모델만큼 좋지 않은 모델을 제공해야 하는 경우가 많습니다.
 - 입력 데이터는 엄격하게 중요하므로 원격 서버에서 해독할 수 없습니다.

 스마트폰이나 임베디드 장치에 Keras 모델을 배포하려면 TensorFlow Lite(www.tensorflow.org/lite)를 사용해야 합니다. ARM64 기반 컴퓨터, 라즈베리 파이 또는 특정 마이크로컨트롤러뿐만 아니라 Android 및 iOS 스마트폰에서 실행되는 효율적인 장치 딥러닝 추론을 위한 프레임워크입니다. Keras 모델을 TensorFlow Lite 형식으로 바로 전환할 수 있는 변환기가 포함되어 있습니다.

---

**브라우저에서 모델 배포**

딥 러닝은 브라우저 기반 또는 데스크톱 기반 자바스크립트 응용 프로그램에서 자주 사용됩니다. 응용 프로그램이 REST API를 통해 원격 모델을 쿼리하는 것이 보통 가능하지만 대신 사용자의 컴퓨터에서 직접 모델을 실행할 수 있는 주요 이점이 있을 수 있습니다. 다음 경우에 이 설정을 사용합니다.

 - 사용자에게 계산 처리를 넘겨 서버 비용을 크게 절감할 수 있습니다.
 - 입력 데이터는 사용자의 컴퓨터 또는 폰에 남아 있어야 합니다.
 - 애플리케이션에는 엄격한 지연 시간 제약이 있습니다. 최종 사용자의 노트북이나 스마트폰에서 실행되는 모델은 서버의 대형 GPU에서 실행되는 모델보다 속도가 느릴 수 있지만, 추가 100ms의 네트워크 왕복 운행 시간은 없습니다.
 - 모델이 다운로드되고 캐시된 후에도 연결 없이 계속 작동하려면 앱이 필요합니다.

물론 모델이 사용자의 노트북이나 스마트폰의 CPU, GPU 또는 RAM을 사용 비중이 작은 경우에만 이 옵션을 사용해야 합니다. 또한 전체 모델이 사용자의 장치에 다운로드되므로 모델에 대해 비밀이 없게 유지해야 합니다. 훈련된 딥러닝 모델이 주어지면 일반적으로 훈련 데이터에 대한 일부 정보를 복구할 수 있다는 사실에 유의해야 합니다. 중요한 데이터에 대해 훈련된 모델은 공개하지 않는 것이 좋습니다.

자바스크립트에서 모델을 배포하기 위해 TensorFlow.js(www.tensorflow.org/js)는 거의 모든 Keras API(원래 WebKeras라는 작업 이름으로 개발됨)뿐만 아니라 많은 하위 수준의 TensorFlow API를 구현한다. 저장된 Keras 모델을 TensorFlow.js로 쉽게 가져와 브라우저 기반 JavaScript 앱 또는 데스크톱 Electronic 앱의 일부로 쿼리할 수 있습니다.

---

**추론 모형 최적화**

사용 가능한 전력 및 메모리(스마트폰 및 임베디드 장치)에 엄격한 제약이 있는 환경이나 대기 시간이 짧은 애플리케이션에 배포할 때 추론을 위해 모델을 최적화하는 것이 중요합니다. TensorFlow.js로 가져오거나 TensorFlow Lite로 내보내기 전에 항상 모델을 최적화해야 합니다.

적용할 수 있는 두 가지 일반적인 최적화 기법이 있습니다.

 - 가중치 가지치기(Weight pruning): 가중치 텐서의 모든 계수가 예측에 동일하게 기여하는 것은 아닙니다. 가장 중요한 항목만 유지하면 모델의 계층에서 매개변수 수를 크게 줄일 수 있습니다. 따라서 성능 지표에서 적은 비용으로 모델의 메모리 및 컴퓨팅 설치 공간을 줄일 수 있습니다. 적용할 가지치기 양을 조정하면 크기와 정확도 사이의 균형을 조정할 수 있습니다.
 - 가중치 정량화(Weight quantization): 딥 러닝 모델은 단일 정밀 부동 소수점(`float32`) 가중치를 사용하여 훈련됩니다. 그러나 가중치를 8비트 부호 정수(`int8`)로 정량화하면 4배 작지만 원래 모델의 정확도에 가까운 추론 모델을 얻을 수 있습니다.
TensorFlow ecosystem은 Keras API와 긴밀하게 통합된 가중치 정리 및 정량화 툴킷(www.tensorflow.org/model optimization)을 포함합니다.

---

### **1.3.3. 모델 모니터링(Monitor your model in the wild)**

이전 과정들이 끝이 아닙니다. 모델을 구축한 후에는 모델의 동작, 새 데이터에 대한 성능, 나머지 애플리케이션과의 상호 작용 및 궁극적으로 비즈니스 메트릭에 미치는 영향을 계속 모니터링해야 합니다.

 - 소비자의 참여가 증가/감소하는 것처럼 **랜덤 A/B 검정(randomized A/B)**을 통해 모델 자체의 영향을 다른 변화로부터 나누는 것을 고려해봐야 합니다. 두 상황의 결과 차이는 모델에서 영향을 받았을 수 있습니다.
 - 생산 데이터에 대한 모델의 예측에 대해 정기적인 수동 감사(manual audit)를 실시합니다. 일반적으로 데이터 주석과 동일한 인프라를 재사용할 수 있습니다. 생산 데이터의 일부를 수동으로 주석을 달도록 전송하고 모델의 예측값을 새 주석과 비교합니다.
 - 수동 감사가 불가능한 경우 사용 설문 조사와 처럼 대체 평가 방법을 고려할 수 있습니다.

---

### **1.3.4 모델 유지(Maintain your model)**

마지막으로, 영원한 모델은 없습니다. **개념 드리프트**에 대해 이미 배웠습니다. 시간이 지남에 따라 생산 데이터의 특성이 바뀌어 모델의 성능과 관련성이 점차 저하됩니다. 음악 추천 시스템의 수명이 몇 주 내로 계산됩니다. 신용카드 사기 탐지 시스템의 경우 며칠이 걸릴 수 있습니다. 이미지 검색 엔진을 위한 최고의 경우 2년입니다. 모델이 출시되자마자 모델을 대체할 다음 모델을 훈련할 준비를 해야 합니다.

 - 생산 데이터 변화 주의하기
 - 데이터를 계속 수집하고 주석을 달 수 있으며 시간이 지남에 따라 주석 파이프라인을 계속 개선할 수 있습니다. 특히 현재 모형에 대해 분류하기 어려운 표본 수집에 특히 주의해야 합니다. 이러한 표본은 성능을 향상시키는 데 가장 큰 도움이 될 수 있습니다.

이것으로 기억해야 할 많은 기계 학습(machine learning)의 일반적인 작업 흐름을 마무리합니다. 이제 기계 학습 프로젝트에 수반되는 전체 스펙트럼이라는 큰 그림을 볼 수 있게 되었습니다. 이 글의 대부분은 모델 개발 부분에 초점을 맞추지만, 이제 전체 워크플로우의 일부분에 불과하다는 것을 알게 되었습니다.

---

## **1.4 요약**

 1. **새 머신러닝 프로젝트를 수행할 때 먼저 문제를 정의합니다.**
    - 최종 목표가 무엇이고, 제약은 무엇인지를 찾아 광범위한 맥락을 이해하기.
    - 데이터 세트를 수집하고 주석을 달 수 있다. 데이터를 자세하게 이해해야 한다.
    - 문제에 대한 성공 여부를 측정하는 방법, 검증 데이터에 대해 어떤 메트릭스를 모니터링할 것인지 선택해야 한다.

 2. **문제를 이해하고 적절한 데이터 세트를 확보하면 모델을 개발합니다.**
    - 데이터 준비
    - 평가 프로토콜 선택: 홀드-아웃 검증, K-겹 검증 등 검증에 사용할 데이터 부분을 고려하여 선택
    - 통계 능력 달성: 기준선을 능가
    - 스케일 업: 과적합 모델을 개발
    - 검증 데이터의 성능에 따라 모델을 정규화하고 하이퍼 파라미터를 조정할 수 있다.

 3. **모델이 준비되고 테스트 데이터에 대해 우수한 성능을 제공하면 이제 배포할 차례입니다.**
    - 먼저 소비자들과 적절한 기대치를 설정합니다.
    - 추론을 위해 최종 모델을 최적화하고 웹 서버, 모바일, 브라우저, 임베디드 디바이스 등의 배포 환경에 모델을 제공합니다.
    - 모델의 생산 성능을 모니터링하고 데이터를 계속 수집하여 차세대 모델을 개발할 수 있습니다.