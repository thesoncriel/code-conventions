# Naming Conventions: Interfaces

TypeScript 의 인터페이스는 다음과 같이 크게 2가지 쓰임새를 가질 수 있습니다.

1. DTO, VO 나 struct 등 데이터 교환 목적
2. 추상화를 위해 클래스(or 객체)가 가지는 기능들만 선언 해 놓은 껍데기.

인터페이스 명칭은 반드시 `Pascal notation` 으로만 작성 합니다.

## 데이터 구조의 목적

공통적으로 `접미어(postfix)`를 가집니다.

이들은 크게 2가지 목적으로 구분할 수 있으며 일반적으로 사용자 계층인 `Client` 와 제공자 계층인 `Server` 로 나뉩니다.

- [Client Side](./interfaces-client.md)
- [Server Side](./interfaces-server.md)

### 작명 시 유의점 - 접두어에 동사(verb) 금지

데이터 구조의 목적을 가진 인터페이스의 명칭 앞부분엔 get, set, put, delete 등과 같은 REST method 에서 쓰일 법한 `동사(verb)`를 붙이지 않습니다.

```ts
// wrong
// 게시물을 가져올 것만 같은 느낌이 듭니다..
interface GetBoardItem {
  boardId: number;
  title: string;
  author: string;
}

// good
// 목적에 부합되는 명칭을 앞에 붙여 주는게 좋습니다!
interface TrendBoardItem {
  // 생략..
}
```
