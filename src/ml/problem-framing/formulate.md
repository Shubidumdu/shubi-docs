# Formulate Your Problem as an ML Problem

해당 섹션에서는 ML 문제를 구성하기 위한 적절한 접근에 대한 가이드를 제공합니다.

1. 문제를 분명히 하라.
2. 간단하게 시작하라.
3. 데이터 소스를 정의하라.
4. 모델에 사용할 데이터를 디자인하라.
5. 어디서 데이터가 올지 결정하라.
6. 쉽게 얻을 수 있는 입력을 결정하라.
7. 학습 능력
8. 잠재적인 편향(bias)에 대해 생각하라.

## 문제를 분명히 하라.

분류와 회귀에도 여러 세부유형들이 존재합니다.
아래 플로우차트를 따라 정확히 어떤 유형의 문제인지를 정의하세요.

<img src="https://developers.google.com/machine-learning/problem-framing/images/FlowChart1.svg?hl=ko" >

<img src="https://developers.google.com/machine-learning/problem-framing/images/FlowChart2.svg?hl=ko" >

우리 문제는 다음과 같이 잘 구성됩니다.

- Binary classification
- Unidimensional regression
- Multi-class single-label classification
- Multi-class multi-label classification
- Multidimensional regression
- Clustering (unsupervised)
- Other (translation, parsing, bounding box id, etc.)

문제를 정확히 한 이후에, 모델이 예측할 내용이 정확히 무엇인지 설명하세요.

이러한 각 요소들을 합쳐서 정리해보면, 문제에 대한 간결한 설명이 됩니다.

<table class="green">
  <tbody><tr><th>Example</th></tr>
  <tr>
    <td>어떤 영상에 대해 다음 3개의 클래스 중 어떤 것에 해당할지 예측하는 단일 레이블 분류 문제를 정의해볼 수 있다.—<code translate="no" dir="ltr">{very popular,<wbr> somewhat popular,<wbr> not popular}</code>— 업로드 후 28일이 된 시점에 대한 예측이다.
    </td>
  </tr>
</tbody></table>


## 간단하게 시작하라.

먼저, 모델링 작업을 간단하게 만들어봅시다. 주어진 문제에 대해 이진 분류 또는 단일 회귀 문제(아니면 둘다)로 정의해보세요. 두 문제들은 도움이 되는 많은 도구와 전문가들의 지원이 있는, 제일 간단한 접근 방식입니다.

그 다음, 작업을 위해서 최대한 간단한 모델을 사용하세요. 간단한 모델은 실행하기도, 이해하기도 쉽습니다. 일단 전체 ML 파이프라인을 갖추고 나면, 간단한 모델에서부터 더 쉽게 발전시켜 나갈 수 있습니다.

<table class="green">
  <tbody><tr><th>Examples</th></tr>
  <tr>
    <td>업로드된 영상이 유명해질 것 같은지 아닌지를 예측한다. (Binary Classification)</td>
  </tr>
    <tr>
    <td>업로드된 영상의 인기도를 28일 이내에 받을 조회수를 기준으로 에측한다. (Regression).</td>
  </tr>
</tbody></table>

실제로 그것을 서비스에 곧장 이용하지 않더라도, 간단한 모델로부터 출발하는 것은 좋은 베이스라인이 됩니다. 사실, 간단한 모델은 오히려 생각보다 더 잘 동작할 겁니다. 단순한 모델은 모델이 제대로 정의되었는지 판단하기 훨씬 쉽습니다. 반면, 복잡한 모델은 훈련시키기도, 이해하기도 훨씬 더 어렵고 느립니다. 그러니 성능을 충분히 향상시킬 정도로 트레이드오프를 갖추지 않는 이상은 이를 단순하게 유지하는 편이 더 좋습니다.

> **주의**: 대부분의 ML은 데이터 쪽에 있습니다. 복잡한 모델에 대한 전체 파이프라인을 실행하는 것은 모델 자체를 반복하는 것보다 어렵습니다.

<img src="https://developers.google.com/machine-learning/problem-framing/images/Barchart.svg?hl=ko">

ML 도입 시에 가장 이득이 큰 시점은 데이터를 처음으로 활용하여 모델을 구축해냈을 때입니다. 추가적인 조정은 여전히 성능의 향상을 일으키지만, 일반적으로 가장 큰 이득을 얻는 시점은 시작점에 있으므로 프로세스를 더 쉽게 만들기 위해 잘 테스트된 방법을 고르는 것이 좋습니다.

## 데이터 소스를 정의하라.

우리의 데이터 레이블에 대해 다음의 질문에 답해봅시다.

- 레이블링된 데이터를 얼마나 많이 갖고 있나?
- 레이블의 출처가 어디인가?
- 해당 레이블은 내리고자 하는 결정과 긴밀하게 연관되어 있는가?

<table class="green">
  <tbody><tr><th>Example</th></tr>
  <tr>
    <td>우리의 데이터 셋은 인기도 데이터 및 영상 설명과 함께 과거에 업로드 된 영상에 대한 100,000 개의 예제로 구성되어 있다.</td>
  </tr>
</tbody></table>

## 모델에 사용할 데이터를 디자인하라.

ML 시스템이 의사 결정을 내리기 위해 사용할 데이터를 정의해야 합니다. (input => output)

<table class="green" label="Example of table with data from a video sharing site.">
  <tbody><tr><th>Title</th><th>Channel</th><th>Upload Time</th><th>Uploader's Recent
    Videos</th><th>Output (label)</th></tr>
  <tr><td>My silly cat</td><td>Alice</td><td>2018-03-21 08:00</td><td>Another cat
    video, yet another cat</td><td>Very popular</td></tr>
  <tr><td>A snake video</td><td>Bob</td><td>2018-04-03 12:00</td><td>None
    </td><td>Not popular</td></tr>
</tbody></table>

각각의 행(row)은 하나의 예측이 수행될 하나의 데이터가 됩니다. 예측이 이루어지는 순간에 사용할 수 있는 정보만을 포함합니다. 각 input은 스칼라 또는 정수/실수/바이트/문자열 등으로 구성된 1차원 리스트가 될 수 있습니다.

만약, input이 1차원 리스트가 아닌 경우, 해당 데이터를 가장 잘 나타낼 방법에 대해 고려해야 합니다. 예를 들자면 :

- 한 input이 사실 상 서로 다른 둘 이상의 항목을 나타내는 경우, 이를 별도의 input으로 분할할 수 있습니다.
- 한 input이 중첩된 [프로토콜 버퍼](https://developers.google.com/protocol-buffers?hl=ko)를 나타내는 경우, 이들의 각 field를 쪼갤 수 있습니다.
- 예외 : 오디오, 이미지, 영상 데이터.. (blob of bytes)

<table class="blue">
  <tbody><tr><th>Tips for audio/image/video data</th><th>Examples</th></tr>
  <tr>
    <td>There may not be explicit inputs.</td>
    <td>The only inputs may be the bytes for the audio/image/video.</td>
  </tr>
    <tr>
    <td>There may be metadata accompanying the image.</td>
    <td>Compression format, object bounding boxes, source</td>
  </tr>
  <tr>
    <td>Your outputs may be simplified for an initial implementation.</td>
    <td>Rather than doing bounding-box object detection, you may create a simple
      binary classifier that learns whether one type of object is present in the
      image or not.</td>
  </tr>
</tbody></table>

## 어디서 데이터가 올지 결정하라.

하나의 행(row)를 구성하기 위해 각 열(column)에 들어갈 값을 만들어줄 데이터 파이프라인을 개발하는 데 얼마나 많은 작업이 필요할지 평가해보세요. 예제 출력을 얻기 어려운 경우, 출력을 다시 고려하여 모델에 다른 출력을 사용할 수 있는지의 여부를 조사해볼 수 있습니다.

모든 입력이 예측 시에 정확히 작성한 형태로 사용할 수 있는지 확인하세요. 예측 시점에 특정한 feature 값을 얻는 것이 어렵다고 판단된다면 해당 특성을 생략해야 합니다.

<table class="green">
  <tbody><tr><th>Example</th></tr>
  <tr>
    <td>We applied the labels <code translate="no" dir="ltr">{very popular,<wbr> somewhat popular,<wbr>
      not popular}</code> to each video that fell within a determined range of
      views and "thumbs ups" and determined keyword descriptions for each video.
      Hand-generating descriptions is not sustainable, so we are considering
      adding a keyword description to the upload form.</td>
  </tr>
</tbody></table>

## 쉽게 얻을 수 있는 입력을 결정하라.

얻기 쉽고, 합리적인 초기 결과를 얻을 수 있을 거라고 판단되는 1-3개의 입력을 선택하세요.

이전에 언급했던 휴리스틱을 구현하는 데 어떤 입력이 유용할까요?

input들을 준비하기 위해 데이터 파이프라인을 개발하는데 드는 엔지니어링 비용과, 모델에 각 input들을 가짐에 따라 예상되는 이득을 고려하세요. 간단한 파이프라인으로 단일 시스템에서 손쉽게 얻을 수 있는 입력에 중점을 두세요. 가능한 최소의 인프라로 시작하세요.

## 학습 능력

우리 ML 모델이 과연 학습할 수 있을까요? 학습에 어려움을 줄 수 있을 것 같은 문제들을 나열해보세요. 예를 들어:

- 데이터 셋에 충분한 양의 레이블이 포함되어 있지 않을 수 있습니다.
- 훈련 데이터에 충분한 예제가 없을 수 있습니다.
- 레이블에 노이즈가 너무 많이 끼어있을 수 있습니다.
- 시스템이 훈련 데이터들을 기억하는 바람에, 새로운 케이스에 일반화시키는 것에 어려움을 겪을 수 있습니다.

<table class="green">
  <tbody><tr><th>Example</th></tr>
  <tr>
    <td>The measure "popular" is subjective based on the audience and
        inconsistent across video genres.
        Tastes change over time, so today's "popular" video might
        be tomorrow's "not popular" video.</td>
  </tr>
</tbody></table>

## 잠재적인 편향에 대해 생각하세요.

많은 데이터 셋들은 어떤 방식으로는 [편향(bias)](https://developers.google.com/machine-learning/glossary?hl=ko#bias_ethics)를 갖습니다. 이러한 편향은 훈련 및 예측에 부정적인 영향을 끼칠 수 있습니다. 예를 들어:

- 편향된 데이터 소스가 여러 문맥 간에 적절하게 해석되지 않을 수 있습니다.
- 훈련 데이터 셋이 모델의 최종 이용자를 대표하지 못할 수 있으며, 이에 따라 부정적인 이용자 경험을 유발할 수 있습니다.

<table class="green">
  <tbody><tr><th>Example</th></tr>
  <tr>
    <td>Since the measure "popular" is subjective, it is possible that the model
      will serve popular videos that reinforce unfair or biased societal views.
    </td>
  </tr>
</tbody></table>