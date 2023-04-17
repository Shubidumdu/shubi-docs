# Machine Learning

## Amazon Rekognition

- ML을 통해 **이미지와 비디오**에서 오브젝트, 사람, 텍스트, 장면을 인식
- **얼굴 분석** 및 **얼굴 검색** - 이용자 검증, 사람 수 카운팅
- familiar face 데이터베이스 생성 또는 유명인사와 비교
- **사례**:
  - labeling
  - 컨텐츠 관리
  - 텍스트 감지
  - 얼굴 감지 및 분석 (성별, 나이 범위, 감정...)
  - 얼굴 검색 및 검증
  - 유명인사 인식
  - pathing (ex. 스포츠 게임 분석)

### Amazon Rekognition - Content Moderation

- 부적절하거나, 원치 않는, 혹은 공격적인 컨텐츠를 감지 (이미지, 비디오)
- 더 안전한 이용자 경험을 보장하기 위해 소셜미디어, 방송 미디어, 광고, e-커머스에서 사용
- **플래그될 항목에 대한 최소 신뢰도 임계값(minimum confidence threshold)를 설정**
- Amazon Augmented AI(A2I) 내에서
- 수동 검토를 위해 민감한 컨텐츠에 플래그 지정
- 규정 준수를 도와줌

## Amazon Transcribe

- **자동으로 speech-to-text 변환을 해줌**
- **automatic speech recognition(ASR)**이라고 불리는 **딥러닝 처리**를 통해 speech-to-text를 빠르고 정확하게 수행할 수 있음
- **Redaction를 통해 자동으로 개인 식별 정보(Personally Identifiable Information ~ PII)를 제거**
- **multi-lingual audio를 통해 언어 자동 감지**
- 사례:
  - 이용자 서비스 전화 트랜스크립트(transcribe)
  - 선택 캡션 및 자막 자동화
  - 완전 검색 가능한 아카이브를 위해 미디어 에셋에 대한 메타데이터 생성

## Amazon Polly

- 딥러닝을 통해 text-to-speech 변환
- 애플리케이션이 말을 할 수 있도록 할 수 있음

### Amazon Polly - Lexicon & SSML

- **Pronunciation lexicons**으로 특정 단어의 발음을 커스터마이징 할 수 있음
  - Stylized words: St3ph4ne => "Stephane"
  - Acronyms: AWS => "Amazon Web Services"
- **Speech Synthesis Markup Language**(**SSML**)로 마크업 가능 - 더 많은 커스터마이징
  - 특정 단어나 문장을 강조
  - 음성 기호대로 발음
  - 숨쉬기, 속삭이기 추가
  - 뉴스 캐스터 화법 사용

## Amazon Translate

- 자연스럽고 정확한 **언어 번역**
- 컨텐츠의 로컬라이징을 하도록 도와줌 - 웹사이트 또는 애플리케이션
  - **국제 이용자**를 위해, 또는 거대한 양의 텍스트를 효율적으로 번역하기 위해 사용

## Amazon Lex & Connect

- **Amazon Lex**: Alexa에 사용되는 것과 동일한 기술
  - Automatic Speech Recognition(ASR)을 사용하여 speech-to-text
  - 텍스트와 호출자(caller)의 의도를 인식하는 Natural Language Understanding
  - 챗봇, 콜센터 봇 생성에 유용
- **Amazon Connect**:
  - 전화를 받고, 응대 플로우를 만들 수 있는 **가상 문의 센터**
  - 다른 CRM 시스템이나 AWS와 호환될 수 있음
  - 선불 결제 없음, 전통적인 문의 센터 솔루션들보다 80% 저렴함

## Amazon Comprehend

- **자연어 처리(NLP ~ Natural Language Processing)**을 위해 사용
- 완전 관리형 서버리스 서비스
- 텍스트에서 인사이트 및 관계 파악
  - 텍스트에서 사용하는 언어
  - 핵심 문구, 장소, 사람, 브랜드, 이벤트 등을 추출
  - 텍스트의 긍정/부정 인식
  - tokenization와 parts of speech(품사)를 통해 텍스트 분석
  - 주제 별로 텍스트 파일 모음을 자동으로 구성
- 사례:
  - 이용자 반응(이메일)을 분석하여 긍정/부정적 경험 여부 판단
  - Comprehend가 발견할 topic을 기반으로 아티클들을 생성 및 그룹화

## Amazon Comprehend Medical

- 비정형 clinical 텍스트 내에서 유용한 정보를 감지하고 반환해줌
  - 의사 소견
  - 퇴원 요약
  - 테스트 결과
  - 케이스 노트
- **Protected Health Information(PHI)를 감지하기 위해 NLP를 사용** - DetectPHI API
- S3 내에 문서 보관, Kinesis Data Firehose로 실시간 데이터 분석, 또는 Amazon Transcribe로 환자 나레이션을 텍스트로 변환하여 Amazon Comprehend Medical이 분석할 수 있도록 함

## Amazon SageMaker

- 개발자 / 데이터 사이언티스트가 ML 모델을 만들 수 있도록 해주는 완전 관리형 서비스
- 일반적으로 ML 모델링을 하는 모든 과정을 한 곳에서 수행하고, 또 서버를 프로비저닝하는 것은 상당히 어렵기 때문

## Amazon Forecast

- ML을 기반으로 매우 정확한 기상 예측을 전달하는 완전 관리형 서비스
- 예시: 우의 판매량 예측
- 데이터 자체를 살펴보는 것보다 50% 정확
- 예측 시간을 몇 달에서 몇 시간으로 단축
- 사례: 제품 수요 예측, 재무 설계, 리소스 계획

## Amazon Kendra

- 머신러닝 기반의 완전 관리형 **문서 검색 서비스**
- 문서 내에서 답을 추출 (text, pdf, HTML, PowerPoint, MS Word, FAQs...)
- 자연어 검색 기능
- 이용자 인터랙션/피드백을 통해 학습하여 선호하는 결과가 나오도록 촉진 (Incremental Learning ~ 점진 학습)
- 검색 결과를 수동으로 미세 조정하는 기능 (데이터 중요도, 신선도, 커스텀...)

## Amazon Personalize

- 실시간으로 개인화된 추천을 전달하는 앱을 구성하기 위한 완전 관리형 ML 서비스
- 사례: 개인화된 상품 추천/re-ranking, 맞춤형 다이렉트 마케팅
  - 사례: 이용자가 정원관리 툴을 샀을 때, 다음에 살만한 제품을 추천
- Amazon.com에도 동일한 기술이 사용됨
- 기존 웹사이트, 애플리케이션, SMS, 이메일 마케팅 시스템과도 호환
- 며칠 내로 구현 가능, 몇 달까지도 안걸림(ML 솔루션을 빌드, 학습, 배포할 필요 없음)
- 사례: 소매점, 미디어 및 엔터테인먼트

## Amazon Textract

- AI와 ML을 통해 스캔된 문서로부터 텍스트, 손글씨, 데이터를 자동으로 추출
- form과 테이블로부터 데이터 추출
- 어떤 타입의 문서든 읽기 및 처리 가능 (PDFs, images, ...)
- 사례:
  - 금융 서비스 (e.g., 청구서, 금융 리포트)
  - 헬스케어 (e.g., 진료 기록, 보험 청구)
  - 공공 부문  (e.g., 세금 신고서, ID 문서, 여권)

## AWS Machine Learning - Summary

- **Rekognition**: 얼굴 감지, 라벨링, 유명인사 인식
- **Transcribe**: speech-to-text (ex. subtitles)
- **Polly**: text-to-speech
- **Translate**: 번역
- **Lex**: 챗봇 빌드
- **Connect**: 클라우드 문의 센터
- **Comprehend**: 자연어 처리
- **SageMaker**: 모든 개발자/데이터 사이언티스트를 위한 머신 러닝
- **Forecast**: 높은 정확도의 기상 예측
- **Kendra**: ML 기반의 검색 엔진
- **Personalize** 실시간 개인 추천
- **Textract**: 문서 내 텍스트 및 데이터 감지
