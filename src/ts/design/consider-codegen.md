# 데이터가 아닌, API와 명세를 보고 타입 만들기

프로젝트를 진행하다보면, 프로젝트 외부에서 비롯된 데이터와 API를 다루게 됩니다.
이 경우 기본적으로 해당 API에서 제공되는 타입 선언(`@types/...`)을 추가하거나, 이로부터 자동 생성되는 타입들을 사용하는 것이 좋습니다.

직접 본인의 경험에 기반하여 타입을 작성할 수도 있지만, 이 경우 모든 예외 케이스를 커버한다고 보장할 수 없습니다.
만약 직접 타입을 작성해야 한다면, 해당 API의 명세 정보를 확인하여 이에 기반하여 작성하는 것이 좋습니다.
