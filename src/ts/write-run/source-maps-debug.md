# 소스맵을 사용하여 타입스크립트 디버깅하기

TS는 런타임에서 직접 실행되지 않습니다. 엄밀히 말하면 TS 뿐 아니라 여러 압축 및 전처리 도구들 모두에 해당하는 내용입니다.
디버거는 런타임 시점에 동작하며, 현재 런타임에 실행 중인 코드가 어떤 과정을 거쳐 만들어졌는지 알지 못합니다.
이렇게 변환된 자바스크립트 코드는 복잡해서 디버깅하기 매우 어렵습니다.

## 소스맵 (source map)

소스맵은 변환된 코드의 위치와 심벌들을 원본 코드의 원래 위치와 심벌들로 매핑합니다.
대부분의 브라우저와 많은 IDE가 소스맵을 지원합니다.

TS 역시 이러한 소스맵에 대한 옵션이 존재합니다.

```json
{
  "compilerOptions": {
    "sourceMap": true
  }
}
```

이후 컴파일을 실행하면 각 `.ts` 파일에 대해 `.js`와 `.js.map` 두 개의 파일을 생성하며, 이 중 `.js.map`이 바로 소스맵에 해당합니다.
소스맵이 `.js` 파일과 함께 있으면 디버거에서 기존에 작성한 `.ts` 파일이 나타납니다.

이제 원하는 대로 브레이크포인트를 설정할 수 있고, 변수를 조사할 수 있습니다.

![sourceMapImg](https://mblogthumb-phinf.pstatic.net/MjAyMDAxMDhfMjE2/MDAxNTc4NDM3OTk1MjEx.DUvBZyim0r6VYxtC6Gt44gzKMbJaEdOewY70DSRyBwQg.YX9FsTW1bPHqWKxCfnP1zEVVUPr_oyPlbIcmrTEUNJ4g.PNG.bunggl/%EC%BA%A1%EC%B2%98.PNG?type=w800)

디버거 좌측의 `app.ts`가 이탤릭 글꼴로 나오는 것은 곧 이것이 웹페이지에 포함된 (컴파일 된) **실제 파일이 아니라는 것**을 의미합니다. 컴파일된 내용이 소스맵을 통해 TS처럼 보이는 것 뿐입니다.

TS 타입체커는 코드 실행 전에 많은 오류를 잡을 수 있지만, 디버거를 대체할 수는 없습니다. 소스맵을 통해 제대로 된 TS 디버깅 환경을 구축하는 것이 필요합니다.

### 주의사항

- TS와 함께 여러 번들러 및 압축기를 사용하고 있다면, 이것이 각자의 소스맵을 생성하게 됩니다. 이상적인 디버깅 환경을 위해선 이들이 원본 TS 소스로 매핑되도록 해야하며, 번들러가 기본적으로 TS를 지원한다면 문제 없겠지만, 그렇지 않다면 번들러가 소스맵을 인식할 수 있도록 추가적인 설정이 필요합니다.
- 프로덕션 환경에 소스맵이 유출되고 있지는 않은지 확인해야 합니다. 소스맵을 통해 공개해서는 안될 내용이 들어 있을 수 있습니다.