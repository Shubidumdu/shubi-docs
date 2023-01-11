# OSI Model

**OSI\(Open Systems Interconnection\)**은 다양한 통신 시스템이 표준 프로토콜을 사용하여 통신할 수 있도록 하는 국제 표준화 기구에 의해 만들어진 개념적인 모델(conceptual model)이다.

OSI 모델은 컴퓨터 네트워킹에 있어 공용 언어로 볼 수 있으며, 이는 커뮤니케이션 체계를 7개의 추상화된 레이어로 쪼개서 보는 개념에서 출발한다.

![OSI Layers](https://www.cloudflare.com/img/learning/ddos/what-is-a-ddos-attack/osi-model-7-layers.svg)

## 왜 OSI 모델이 중요할까?

현대의 인터넷은 사실 엄격하게 OSI 모델을 따르지 않음에도 불구하고, OSI 모델은 여전히 네트워크 문제에 대한 트러블슈팅에 매우 용이하다.

노트북에서 인터넷 연결이 안되는 문제, 수천명의 이용자가 사용하는 웹사이트가 다운되는 문제 등등, 이러한 네트워크와 관련된 문제를 쪼개서 문제의 원인을 정확히 파악하는 데에 OSI 모델이 도움을 줄 수 있다. 만약, 어떤 문제가 모델 내 하나의 구체적인 레이어에 국한된 점이라는 것을 파악한다면, 그 외의 불필요한 작업들은 피할 수 있다.

## 7. The application layer

![Application Layer](https://cf-assets.www.cloudflare.com/slt3lc6tev37/koKt5UKczRq47xJsexfBV/c1e1b2ab237063354915d16072157bac/7-application-layer.svg)

이용자와 직접적으로 데이터를 통해 상호작용하는 유일한 레이어다. 웹 브라우저나 이메일 클라이언트와 같은 소프트웨어 애플리케이션들이 이 애플리케이션 레이어에 의존하여 통신을 시작한다.

단, 클라이언트 소프트웨어 애플리케이션이 애플리케이션 레이어의 일부에 해당하는 것은 아니라는 점을 기억하자. 오히려 애플리케이션 레이어는 소프트웨어가 사용자에게 의미있는 데이터를 전달하기 위해 의존하는 프로토콜과 데이터 조작에 대한 책임을 갖는다.

애플리케이션 레이어 프로토콜은 HTTP 뿐만 아니라 SMTP(Simple Mail Transfer Protocol ~ 이메일 커뮤니케이션을 가능하게 하는 프로토콜 중 하나)를 포함한다.

## 6. The Presentation Layer

![Presentation Layer](https://cf-assets.www.cloudflare.com/slt3lc6tev37/60dPoRIz0Es5TjDDncEp2M/7ad742131addcbe5dc6baa16a93bf189/6-presentation-layer.svg)

애플리케이션 레이어에서 사용될 데이터를 준비하는 역할을 하는 레이어다. 즉, 프레젠테이션 레이어는 애플리케이션이 소비할 수 있는 데이터를 제공한다. 해당 레이어는 변환(translation), 암호화(encryption), 데이터 압축(compression of data)를 담당한다.

> translation - 디바이스마다 다른 인코딩 방식으로 인코딩된 데이터를 변환
>
> encryption - 암호화된 연결을 통해 통신하는 경우, 암호화/복호화를 담당
>
> compression - 애플리케이션 레이어를 통해 넘겨받은 데이터를 아래 층 레이어로 전달하기 전에 압축

## 5. The Session Layer

![Session Layer](https://cf-assets.www.cloudflare.com/slt3lc6tev37/6jFRnaZSuIMoUzSotZXYbG/cc7a47d2b3f8d3e77b9ffbdb8b8d5280/5-session-layer.svg)

두 디바이스 간의 통신(communication)을 열고 닫는 역할을 하는 레이어다. "세션"은 통신이 열리고 닫히는 시간을 의미한다. 세션 레이어는 세션이 교환되는 모든 데이터를 전송할 수 있을 정도로 세션이 열려있는지 확인하고, 리소스 낭비를 방지하기 위해 세션을 빠르게 닫는다.

또한, 데이터 전송을 체크 포인트를 통해 동기화해주는 역할도 수행한다.

## 4. The Transport Layer

![Transport Layer](https://cf-assets.www.cloudflare.com/slt3lc6tev37/1MGbIKcfXgTjXgW0KE93xK/64b5aa0b8ebfb14d5f5124867be92f94/4-transport-layer.svg)

두 디바이스 간의 E2E 통신을 담당하는 레이어다. 세션 레이어로부터 데이터를 넘겨 받아 이것을 세그먼트(segment)라고 하는 덩어리들로 쪼갠 다음 아래 층의 레이어로 전송한다. (Segmentation)

넘겨받는 디바이스 쪽의 해당 레이어에서는 이렇게 세그먼트로 쪼개진 데이터를 다시 소비가 가능한 형태로 재조립(reassembly)하는 역할을 수행한다.

트랜스포트 레이어는 Flow Control과 Error Control의 역할도 수행한다.

> Flow Control - 빠른 연결을 가진 송신자가 느린 연결을 가진 수신자를 압도하지 않도록 하기 위한 최적의 전송 속도를 결정하는 것
>
> Error Control - 데이터 수신이 완료되었는지에 대해 확인하고, 그렇지 못한 경우 재전송을 요청

## 3. The Network Layer

![Network Layer](https://cf-assets.www.cloudflare.com/slt3lc6tev37/76JgEjycZl12c90UByKfJA/d6578bcd7b151c489e61f42227a45713/3-network-layer.svg)

두 개의 다른 네트워크 간의 데이터 전송을 용이하게 하는 역할을 하는 레이어다. 만약 두 개의 디바이스가 동일한 네트워크 상에서 통신한다면, 네트워크 레이어는 필요없다.

네트워크 레이어는 트랜스포트 레이어로부터 넘겨 받은 세그먼트를 패킷(packet)이라고 하는 더 작은 단위로 쪼갠다. 수신하는 측에서는 거꾸로 이러한 패킷을 재조립해준다.

네트워크 레이어는 또한 데이터가 대상에 도달하기 위한 최적의 물리적 경로를 찾는 역할도 수행하는데, 이를 라우팅(routing)이라고 한다.

## 2. The Data Link Layer

![Data Link Layer](https://cf-assets.www.cloudflare.com/slt3lc6tev37/3MR4mPOwaos80t1annw7BG/8ea1c59ccfa1baf6e9738773daa30450/2-data-link-layer.svg)

데이터 링크 레이어는 네트워크 레이어랑 매우 유사하다. *동일한* 네트워크의 두 디바이스 간 데이터 전송을 용이하게 한다는 점만 빼면.

데이터 링크 레이어는 네트워크 레이어로부터 패킷을 받아 이것을 프레임(frame)이라고 불리는 더 작은 조각들로 쪼갠다.

데이터 링크 레이어도 인트라 네트워크(intra-network) 통신 내에서의 flow control과 error control의 처리를 담당한다.

## 1. The Physical Layer

![Physical Layer](https://cf-assets.www.cloudflare.com/slt3lc6tev37/3m1ZkcaaBYHoodrEO3brv2/2819c4db294631b5753cd55de0c01bd9/1-physical-layer.svg)

케이블, 스위치와 같은 데이터 전송과 관련된 물리적 장비가 포함되는 레이어다.

데이터가 비트스트림(bit stream ~ 1과 0의 스트링)으로 변환되는 레이어로, 두 디바이스 간 피지컬 레이어는 신호 규칙(signal convention)이 일치해야 양 디바이스 에서의 1과 0을 구별할 수 있게 된다.
