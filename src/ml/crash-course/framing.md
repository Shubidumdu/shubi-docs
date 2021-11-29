# Framing

지도 학습(Supervised Learning)이란 무엇일까요? 결론적으로는, 아래를 의미합니다.
- 이전에 보지 못한 데이터에 대해 유용한 예측을 생성하는 방법을 배우는 ML 시스템

## Labels (레이블)

**레이블**은 **우리가 예측하려는 것**입니다. 단순 선형 회귀에서 `y` 변수에 해당하는 것이죠. 레이블에는 다음과 같은 것들이 될 수 있습니다.

- 쌀의 추후 가격 예측
- 사진에 찍힌 동물의 종류
- 오디오 클립의 의미


## Features (피쳐)

**Feature**는 입력 변수입니다. 단순 선형 회귀에서 `x` 변수에 해당하는 것입니다.
가장 단순한 형태의 ML 프로젝트는 단 하나의 피쳐만 사용함으로써 이루어집니다.
반면, 엄청 섬세한 형태의 ML 프로젝트는 수십만개의 피쳐가 존재할수도 있죠.

```
x1,x2,...xN
```

스팸 메일 분류기를 예시로 들자면, 피쳐는 아래와 같은 것들이 될 수 있습니다.

- 이메일 텍스트에 들어간 단어들
- 발신자 주소
- 이메일이 보내진 시각
- "one weird trick" 이라는 문구가 포함되었는지의 여부

## Examples (예시)

**Example**은 데이터의 구체적인 한 예시입니다. <b>x</b>로 표현되기도 합니다. (굵은 글씨로 되었다는 점을 유의하세요.) 이러한 Example은 또 두개로 분류될 수 있습니다.

- Labeled examples
- Unlabeled examples

레이블 처리된 예시의 경우 Feature들과 Label을 모두 갖습니다. 

```
labeled examples: {features, label}: (x, y)
```

이러한 레이블 처리된 예시들은 모델을 <b>Train(훈련)</b> 시키기 위해 사용됩니다. 우리의 스팸 분류기 예시에서, "스팸인지 아닌지"에 대해 명시되어있는 레이블 처리된 예시들이 곧 레이블 처리된 예시들이 됩니다.

예를 들어, 아래 5줄의 예시로 구성된 테이블은 켈리포니아의 집값 정보를 담고있는 데이터셋입니다.

<table>
  <tbody><tr>
    <th>housingMedianAge<br>(feature)</th>
    <th>totalRooms<br>(feature)</th>
    <th>totalBedrooms<br>(feature)</th>
    <th>medianHouseValue<br>(label)</th>
  </tr>

  <tr>
    <td>15</td>
    <td>5612</td>
    <td>1283</td>
    <td>66900</td>
  </tr>
  <tr>
    <td>19</td>
    <td>7650</td>
    <td>1901</td>
    <td>80100</td>
  </tr>
  <tr>
    <td>17</td>
    <td>720</td>
    <td>174</td>
    <td>85700</td>
  </tr>
  <tr>
    <td>14</td>
    <td>1501</td>
    <td>337</td>
    <td>73400</td>
  </tr>
  <tr>
    <td>20</td>
    <td>1454</td>
    <td>326</td>
    <td>65500</td>
  </tr>
</tbody></table>

반면 레이블이 없는 예시의 경우 Feature는 갖고 있지만 Label은 없습니다.

```
unlabeled examples: {features, ?}: (x, ?)
```

아래는 위와 동일한 집값 데이터이지만, Label에 해당하는 `medianHouseValue`가 존재하지 않습니다.

<tbody><tr>
    <th>housingMedianAge<br>(feature)</th>
    <th>totalRooms<br>(feature)</th>
    <th>totalBedrooms<br>(feature)</th>
  </tr>
  <tr>
    <td>42</td>
    <td>1686</td>
    <td>361</td>
  </tr> 
  <tr>
    <td>34</td>
    <td>1226</td>
    <td>180</td>
  </tr>
  <tr>
    <td>33</td>
    <td>1077</td>
    <td>271</td>
  </tr>
</tbody>

일단 우리가 레이블 처리된 모델을 훈련시키고 나면, 레이블이 없는 예시들에 대해서도 예측을 수행할 수 있습니다. 스팸 분류기에서 레이블이 없는 데이터는 사람들이 아직 레이블을 처리하지 않은 새로운 이메일이 되겠죠.

## Models

**모델**은 Feature와 Label 간의 관계를 의미합니다. 예를 들어, 스팸 분류기 모델은 일부 Feature들은 스팸과 강하게 연관이 있다고 판단할 것입니다. 모델이 사용되는 두 단계에 대해서 살펴봅시다.

- **Training(훈련)**은 모델을 만들고, **가르치는 것**을 의미합니다. 다시 말해, 모델에게 레이블처리된 예시를 보여주고, 점차 모델에게 Feature와 Label 간의 관계를 가르치는 것입니다.
- **Inference(추론)**는 훈련이 완료된 모델을 레이블이 되어있지 않은 예시에 적용함을 의미합니다. 다시 말해, 유용한 예측(`y'`)을 수행할 수 있는 모델을 사용하는 것이죠. 앞선 경우를 예로 들자면, 레이블이 없는 예시에 대해 집값에 해당하는 Label인 `medianHouseValue`를 예측하는 것을 의미합니다.

## Regression vs. Classification

Regression(회귀)는 모델이 연속성을 띄는 값을 예측하는 것을 의미합니다. 예를 들어, 회귀 모델의 경우 아래와 같은 문제들에 대한 답을 예측합니다.

- 켈리포니아의 집값은 얼마일까?
- 이 광고를 이용자가 클릭할 가능성은 얼마나 될까?

Classification(분류)는 모델이 분리된 값에 대한 예측을 하는 것을 의미합니다. 예를 들어, 분류 모델은 아래와 같은 문제들에 대한 답을 예측합니다.

- 수신한 이메일이 스팸일까 아닐까?
- 사진의 동물은 개일까 고양이일까?