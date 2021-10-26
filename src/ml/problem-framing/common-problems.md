# Common ML Problems

일반적인 관점에서, 머신 러닝이란 **모델**이라 불리는 하나의 소프트웨어를 훈련시키는 과정입니다. 모델은 데이터 셋을 사용해 유용한 예측을 수행합니다.

종종, 사람들은 머신러닝을 *지도학습*과 *비지도학습*이라는 두가지 패러다임으로 구분합니다. 하지만, 실제로 머신러닝은 이 두가지 학습 방식이 복합적으로 어우러진다고 보는 편이 더 정확합니다. 여기서는 간단함을 위해, 좀 더 양쪽에 극단적인 케이스를 다루고자 합니다.

## 지도 학습

지도 학습은 **label**처리된 트레이닝 데이터들을 제공받는 머신러닝의 유형입니다.

예를 들어, 아래와 같은 식물들의 데이터를 통해, 각 식물이 어떤 종에 속하는지를 맞추는 것과 같죠.

<table>
<thead>
<tr>
<th>Leaf Width</th>
<th>Leaf Length</th>
<th>Species</th>
</tr>
</thead>
<tbody>
<tr>
<td>2.7</td>
<td>4.9</td>
<td>small-leaf</td>
</tr>
<tr>
<td>3.2</td>
<td>5.5</td>
<td>big-leaf</td>
</tr>
<tr>
<td>2.9</td>
<td>5.1</td>
<td>small-leaf</td>
</tr>
<tr>
<td>3.4</td>
<td>6.8</td>
<td>big-leaf</td>
</tr>
</tbody>
</table>

여기서 잎의 너비와 길이는 **Features**입니다. 그리고 우리가 맞추어야 하는 종(Species)는 곧 **Label**이 되죠. 아마 실제로는 훨씬 더 많은 Feature들이 존재할테지만, 여전히 Label은 한개일겁니다. 그리고, 이 Label이란 개념은 반드시 "정답"과 같은 느낌으로, 명확해야 합니다.

위에서는 단순히 4개의 예시만 다루었지만, 실제로는 훨씬 더 많은 데이터가 존재할 겁니다. 만약 데이터셋이 다음과 같은 그래프를 그려낸다고 생각해봅시다.

<img src="https://developers.google.com/machine-learning/problem-framing/images/Graph1.svg?hl=ko">

지도학습에서는, 각 데이터셋의 케이스에 대한 Feature와 Label들을 알고리즘에 전달하는 **Training**이라 불리는 과정을 거칩니다. 이러한 과정 속에서, 알고리즘은 점차적으로 Feature들과 Label 간의 상관관계를 파악하게 되며, 이러한 상관관계가 곧 **모델**이 됩니다. 머신러닝에서 대부분 이러한 모델은 굉장히 복잡한 형태지만, 당장에는 다음과 같이 선 하나로 표현될 수 있는 모델이 있다고 하겠습니다.

<img src="https://developers.google.com/machine-learning/problem-framing/images/Graph2.svg?hl=ko">

이제 모델은 난생 처음보는 케이스들에 대해서 나름의 예측을 할 수 있게 됩니다.

<img src="https://developers.google.com/machine-learning/problem-framing/images/Graph3.svg?hl=ko">


## 비지도 학습 

비지도 학습에서의 목표는 데이터 내에 존재하는 **의미있는 패턴**을 발견하는 것입니다. 이를 위해서 기계는 별도의 Label이 존재하지 않는 데이터로부터 배워야합니다. 다시 말해, 모델은 각 데이터 조각들을 어떻게 분류해나가고, 어떻게 처리되어야 하는지에 대한 별도의 힌트를 전달받지 못합니다.

아래 그래프에는 별도의 Label이 없습니다. 그렇기 때문에 모두 초록색 동그라미로 표기됩니다.

<img src="https://developers.google.com/machine-learning/problem-framing/images/Graph4.svg?hl=ko" />

이처럼 Label이 없는 데이터셋에 아래와 같이 선을 긋는 것은 무의미합니다. 여전히 줄의 양쪽에는 동일한 초록 동그라미들이 있을 뿐입니다.

<img src="https://developers.google.com/machine-learning/problem-framing/images/Graph5.svg?hl=ko" />

한번 다르게 접근해봅시다. 여기 두 **Cluster**(군집)이 있습니다. 이 군집들은 무엇을 나타낼까요? 아마 대답하기는 어려울겁니다. 때때로 모델은 우리가 배우지 않기를 바라는 데이터에 대한 패턴을 찾아내기도 합니다. 스테레오타입(Stereotypes)과 편향(Bias)같은 것들이죠.

<img src="https://developers.google.com/machine-learning/problem-framing/images/Graph6.svg?hl=ko" />

그럼에도 부룩하고, 새로운 데이터가 나타났을 때, 그것이 현재 알고 있는 군집에 적합한 데이터라면 우리는 쉽게 이를 분류할 수 있습니다. 그런데, 만약 우리가 난생 처음 보는 군집에 해당하는 데이터라면 어떨까요?

<img src="https://developers.google.com/machine-learning/problem-framing/images/Graph7.svg?hl=ko" />

> 참고 : 사실, 클러스터링(군집화)는 비지도 학습의 유일한 형태가 아닙니다. 비지도 학습에는 여러 유형이 있고, 군집화는 그 중 가장 일반적인 방식일 뿐입니다.

### 강화 학습

머신 러닝의 추가적인 갈래로 강화 학습(Reinforcement Learning)이 있습니다. 이는 머신러닝의 다른 유형들과 구분됩니다. 기본적으로 이는 Label을 가진 데이터를 수집하지 않습니다. 만약 기계에게 정말 간단한 비디오게임을 플레이하게 만들고, 절대 지지 않게끔 가르치려고 한다고 해봅시다. 이는 기계에게 게임을 하되, 게임오버에 도달하지 말라고 말하는 것과 같습니다. 기계는 학습 중에 **reward** 함수라 불리는 것을 통하여 각 태스크에 대한 보상을 전달받습니다. 강화 학습을 거치면서, 이들은 실제로 사람을 뛰어넘는 수준으로 빠르게 학습할 수도 있습니다.

기본적으로 강화 학습은 수많은 데이터를 요구하지 않습니다. 그러나 좋은 reward 함수를 디자인하는 것은 어려우며, 그렇기 때문에 지도 학습보다 덜 안정적이고, 예측이 어렵습니다. 추가적으로, 기계가 게임과 같은 것들을 플레이 해나가며 상호작용을 통해 데이터를 생산해나갈 방법 자체도 고려해야합니다. 만약, 우리가 실 생활에서나, 혹은 VR 세계에서 강화학습을 적용해나가고자 한다면, 대단히 어려운 과정이 될 것입니다. 강화 학습도 머신러닝 분야에서 굉장히 활발하게 연구되고 있으나, 여기에서는 별도로 자세히 알아보지는 않도록 하겠습니다.

## 머신러닝 문제의 유형

머신러닝 문제는 *예측이 어떤 식으로 이루어지느냐*에 따라 여러가지로 구분됩니다. 아래는 일반적인 지도/비지도 학습의 예시들을 다루고 있습니다.

<table class="blue">
  <tbody><tr><th>Type of ML Problem</th><th>Description</th><th>Example</th></tr>
  <tr>
    <td>Classification</td>
    <td>Pick one of N labels</td>
    <td>Cat, dog, horse, or bear</td>
  </tr>
    <tr>
    <td>Regression</td>
      <td>Predict numerical values</td>
      <td>Click-through rate</td>
  </tr>
    <tr>
    <td>Clustering</td>
      <td>Group similar examples</td>
      <td>Most relevant documents (unsupervised)</td>
  </tr>
   <tr>
    <td>Association rule learning</td>
      <td>Infer likely association patterns in data</td>
      <td>If you buy hamburger buns, you're likely to buy hamburgers
        (unsupervised)</td>
  </tr>
    <tr>
    <td>Structured output</td>
      <td>Create complex output</td>
      <td>Natural language parse trees, image recognition bounding boxes</td>
  </tr>
      <tr>
    <td>Ranking</td>
      <td>Identify position on a scale or status</td>
      <td>Search result ranking</td>
  </tr>
</tbody></table>