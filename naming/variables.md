# Naming Conventions: Variables

변수명 작성 시 누구나 공감하고 인지가 가능한 명칭으로 사용하는 것이 좋습니다.

본 문서는 코딩 시 빈번히 발생되는 변수명에 대한 작성 가이드를 알려 드립니다.

## 기본 작성 방법

변수는 특별한 사유가 없다면 `camel-case` 로 작성합니다.

그리고 변수 선언 문은 로직 진행 중 값이 변동된다면 `let` 을, 그렇지 않다면 `const`를 이용합니다.

`var`는 정적 페이지를 만들거나 외부 스크립트 삽입등의 이유가 아니면 절대 사용하지 않습니다.

```ts
// 이후에 변경이 있습니다.
let complexCompanyName = '고위드';

if (isProjectCompleted()) {
  complexCompanyName = '고위드 완성!';
}

// 변경이 없을 때
const snowWhiteSmile = '미소는 아름답습니다 :)';

// Nope! 쓰지 않습니다.
var myNumber = 1234;
```

## 복수형(Plural)

기본적으로 복수형으로 표현이 가능한 변수는 그대로 사용될 수 있습니다. (예: user -> users, product -> products)

다만, 여러 사정으로 적절한 네이밍이 고민되거나 목록형 자료임을 보다 명확히 명시하고 싶은 경우 `items`를 사용합니다.

그리고 이들에서 파생된 각각의 항목은 `item`으로 사용합니다.

### 규칙의 의의

보통 배열에 대한 변수명이나 속성명(prop) 이용 시 아래와 같이 막연히 `data` 나 `list` 등을 사용하는 경우가 있습니다.

이 때 `items` 와 `item` 을 사용하여 변수명에 일관성을 높이는 것을 목적으로 합니다.

```ts
// 아래 상수들은 서로 다른 scope 에서 선언되었다 가정합니다.

// 이전에 이랬다면..
const data = response.data; // 배열형 자료를 받음
const list = [1, 2, 3, 4]; // 배열형 자료를 만듬

// 이젠 이렇게!
const items = response.data; // 배열형 자료를 받음
const items = [1, 2, 3, 4]; // 배열형 자료를 만듬
```

한편, 타입을 알고 있으며 복수형으로 하는 것이 자연스럽다면 그 것을 이용하는 것 또한 좋습니다.

그리고 아래 예시처럼 필요 시 접미어(postfix) 활용도 좋습니다.

```ts
// 이것도 좋고..
const products = response.data;
const books = getBooks();

// 이것도 좋습니다!
const productItems = response.data;
const bookItems = getBooks();

// but, 이건 안됩니다!
const productData = response.data;
const bookData = getBooks();
```

다만, 아래와 같이 너무 일반적이라 목적을 알 수 없는 용도는 주의 합니다.

```ts
// 무엇을 위한 목적이죠??
const numbers = [1, 2, 3, 4];

// 목적을 명확히 구분 해 줍시다.
const pageNumbers = [1, 2, 3, 4];
```

### 사용 예시

아래는 일반적인 함수 내 수행 예시 입니다.

```ts
function convert(items: DemoItem[]): ChangedItem[] {
  return items.map((item) => ({
    name: item.keyName,
    currency: parseInt(item.value),
  }));
}

function hasChanged() {
  const items = getChangedItems();

  return items.length > 0;
}

// 체이닝 예시
const items = [1, 2, 3, 4, 5];

// Array 에서 지원하는 체이닝 메서드에서 item 으로 사용
const evenItems = items
  .map((item) => item + 1)
  .filter((item) => item % 2 === 0);
```

아래는 컴포넌트의 props 예시 입니다.

```tsx
interface Props {
  items: UserItem[];
  show?: boolean;
}

export const SampleUserList: FC<Props> = ({ items, show }) => {
  return (
    <ul>
      {items.map((item, idx) => (
        <li key={`sample-user${idx}`}>{item.name}</li>
      ))}
    </ul>
  );
};
```

## 객체 필드명에 data 를 남발하지 마세요

data 라는 이름의 필드명은 굉장히 자주 쓰입니다.

서버에서 전달되는 응답 자료에는 어쩔 수 없지만, 저희는 필요에 따라 data 를 지양하기로 하였습니다.

### data.data.data ...

객체의 필드를 연쇄적으로 접근하다 보면 `data.data.data ...` 와 같은 형태를 목격하게 됩니다!

레거시 코드가 아닌 한, 새로 작성하게 되는 코드는 `data` 라는 단일 필드명은 짓지 않습니다.

부득이하게 data 로 사용해야만 한다면, 대신 아래와 같이 그 data 의 `interface model naming convention` 을 이용합니다.

> interface model naming convention 은 추 후 다룰 예정 입니다!

참고로 아래는 좀 과장된 경우 입니다. 😅

```ts
// 아래와 같은 모델이 있다 가정

interface PetShopInfoModel {
  name: string;
  location: string;
  specific: string;
  desc: string;
  likeCounts: number;
}
```

```ts
// bad

interface DataState {
  data: PetShopInfoModel;
}

interface ResultState {
  data: DataState;
}

const data: ResultState = {
  data: {
    data: {
      name: '포메 사랑',
      location: '대한민국 포메시',
      specific: '포메 많음',
      desc: '포메포메포메...',
      likeCounts: 10000000,
    },
  },
};

// WTF ?!
const petShop = data.data.data;
```

```ts
// good

interface DataState {
  petShopInfo: PetShopInfoModel;
}

interface ResultState {
  state: DataState;
}

const result: ResultState = {
  state: {
    petShopInfo: {
      name: '포메 사랑',
      location: '대한민국 포메시',
      specific: '포메 많음',
      desc: '포메포메포메...',
      likeCounts: 10000000,
    },
  },
};

// -ㅂ-)b
const petShop = result.state.petShopInfo;
```

### 규칙의 의의

상기 복수형 규칙과 유사하게 막연히 `data` 명칭으로 사용하는 것을 제한하기 위함 입니다.

그럼으로써 코드 작성 시 각 필드 명칭을 좀 더 의미 있게 구성하여 다른 동료들도 읽기 쉬운 코드가 될 것입니다.
