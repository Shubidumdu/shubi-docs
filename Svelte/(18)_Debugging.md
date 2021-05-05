# Debugging

## The @debug tag

때때로, 애플리케이션을 이용하는 중에 데이터 흐름을 체크하는 과정이 필요하다.

일반적으로 이는 `console.log(...)`을 통해 처리되곤 하는데, 만약 실행을 멈추고 해당 값을 확인하고자 한다면, `{@debug value1, value2, ...}` 태그를 사용할 수 있다.

```svelte
{@debug user}

<h1>Hello {user.firstname}!</h1>
```

이제 `user`의 값이 변경될 떄마다 debugger가 동작할 것이다.
