# Descending into ML

## Linear Regression

귀뚜라미가 추운 날보다 더운 날에 더 자주 운다는 것은 익히 알려진 사실입니다. 지난 몇년간, 전문가들과 과학자들은 온도와 귀뚜라미의 울음 주기에 대한 데이터를 수집해왔습니다. 해당 데이터를 통해 둘 간의 상관 관계를 파악해봅시다.

먼저, 플롯을 통해 데이터에 대해 시각적으로 살펴봅시다.

<img src="https://developers.google.com/machine-learning/crash-course/images/CricketPoints.svg?hl=ko" />

#### 그림 1. 분당 울음횟수 vs. 섭씨 온도

예상한 대로, 온도가 올라갈 수록 귀뚜라미의 울음 주기도 많아집니다. 이것이 둘 사이의 상관관계 일까요? 맞습니다. 여기에 하나의 선을 그어 보면 그 관계를 더 뚜렷하게 알 수 있습니다.

<img src="https://developers.google.com/machine-learning/crash-course/images/CricketLine.svg?hl=ko" />

#### 그림 2. 선형 관계

네, 맞습니다. 선이 모든 점들을 꿰뚫지는 않고 있죠. 하지만 위에 그은 선은 두 변수 간의 상관관계를 뚜렷하게 보여줍니다. 이걸 방정식으로 나타내본다면, 아래와 같은 형태겠죠.

$$ y = mx + b $$

- $y$는 섭씨 온도입니다. -> 예측하고자 하는 값이죠.
- $m$는 선의 기울기입니다.
- $x$는 귀뚜라미의 분당 울음 횟수입니다. -> 입력 Feature로 주어지는 값이죠.
- $b$는 y-절편(intercept)입니다.

이걸 머신러닝 컨벤션에 따라 작성한다면 약간 다르게 아래와 같은 방정식이 됩니다.


$$ y' = b + w_1x_1 $$

- $y'$은 예측된 레이블입니다. (예상되는 결과)
- $b$는 bias(편향)입니다. 종종 $w_0$로 불리기도 합니다.
- $w_1$는 Feature1의 weight(가중치)입니다.
- $x_1$은 Feature(피쳐)입니다. (알려진 입력)

새로운 분당 귀뚜라미 울음 횟수값인 $x_1$에 대한 온도 예측 $y'$을 알기 위해서는 단순히 $x_1$ 값을 위 모델에 집어넣기만 하면 됩니다.

반면, 우리가 지금껏 살펴본 모델은 하나의 Feature만을 갖는 매우 단순한 모델입니다. 실제로 더 복잡한 모델은 더 많은 Feature들에 의존합니다. ($w_1$, $w_2$, ...) 예를 들어, Feature가 3개로 늘어난다면 아래와 같은 모델이 만들어지게 되는거죠.

$$ y' = b + w_1x_1 + w_2x_2 + w_3x_3 $$

## Training and Loss


모델을 **훈련**시키는 것은 레이블 처리된 예시들로부터 적절한 가중치(Weight)와 편향(Bias)를 배우게 하는 것을 의미합니다. 지도 학습에서, 머신러닝 알고리즘은 수많은 예시들을 살펴보고 loss(손실)을 줄여나가면서 모델을 완성하게 됩니다. 이러한 과정을 Empirical Risk Minimization(경험적 위험 최소화)라고 합니다.

loss는 좋지 않은 예측을 한 경우에 대한 페널티와 같은 개념입니다. 즉, **loss**는 하나의 예시에 대한 모델의 예측이 <b>얼마나 별로였나?</b>를 나타내는 값이죠. 만약 모델의 예측이 완벽했다면, loss는 0이 됩니다. 모델을 훈련시키는 것의 목적은 모든 예시에서 평균적으로 loss를 갖도록 하는 여러 가중치들과 편향을 찾아나가는 것입니다. 예를 들어, 아래의 그림3은 좌측에 loss가 큰 모델과 우측의 loss가 적은 모델을 각각 그래프로 나타내고 있습니다.

- 화살표선은 loss를 나타냅니다.
- 파란색 선은 prediction을 의미합니다.

<img src="https://developers.google.com/machine-learning/crash-course/images/LossSideBySide.png?hl=ko" >

#### 그림3. loss가 큰 왼쪽 모델; loss가 작은 오른쪽 모델.

왼쪽의 각 화살표선들이 오른쪽의 각 화살표선보다 훨씬 길다는 것을 유의하세요. 이런 경우, 우측 모델이 명백히 **더 나은 모델**이 됩니다.

이쯤 되면 수학적으로 어떻게 loss를 계산하는지에 대한 <b>Loss Function(손실 함수)</b>가 궁금할 겁니다.

### Squared Loss: 가장 인기있는 손실 함수

우리가 살펴볼 손실 함수는 $L_2$ loss 라고도 알려진 Squared Loss입니다. 이건 식으로 나타내자면 아래와 같습니다. $y$는 실제 Label의 값을, $y'$은 $x$에 대한 예측값 $prediction(x)$을 의미합니다.

$$ (y - prediction(x))^2 $$

평균제곱오차(Mean Square Error: MSE)는 전체 데이터셋의 각각에 대한 Squared Loss의 평균입니다. MSE를 계산하려면, 각각의 예시에 대한 Squared Loss를 전부 더하고, 예시의 갯수만큼 나누어주면 되죠.

$$  MSE = \frac{1}{N} \sum_{(x,y)\in D} (y - prediction(x))^2  $$

- $(x, y)$ 는 아래를 의미합니다.
  - $x$는 모델의 예측에 사용될 Feature들의 집합입니다. (ex. 분당 귀뚜라미 울음 횟수, 나이, 성별..)
  - $y$는 예시의 Label입니다. (ex. 온도)
- $prediction(x)$는 여러 개의 Feature $x$와 가중치($w$), 편향($b$)의 조합으로 이루어진 함수입니다.
- $D$는 레이블 처리 된 많은 예시들을 담고 있는 데이터셋입니다. 이는 $(x, y)$의 쌍으로도 나타낼 수 있습니다.
- $N$은 $D$에 속하는 데이터 예시들의 갯수입니다.

MSE는 머신러닝에서 일반적으로 사용되기는 하지만, 모든 상황에서 유일하게 사용되는 최상의 손실 함수는 아닙니다.