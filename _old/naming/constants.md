# Naming Conventions: Constants

상수는 변하지 않는 값으로써 개발 시 변수만큼 자주 쓰이게 됩니다.

이 때 상수명 작성 시 빈번히 발생되는 패턴을 아래와 같이 정리 해 둡니다.

> 참고: 업무적으로 쓰이는 상수
>
> 보통 업무 로직을 구성할 때는 상수 보단 변수가 더 많이 쓰입니다.
>
> 다만, eslint 의 권장사항으로 내부에 변동사항이 없는 변수는 `let` 이 아닌 `const`로 선언하게 되는데, 이런것은 본 문서의 언급 대상이 아닙니다. 🙂

## Upper Case

일반적인 상수명은 기본적으로 `Upper-Case` 로 통일 합니다.

## 단일 상수

단일 상수는 그 타입이 `number`, `string` 등 원시형(primitive type)에 가까운 것들을 사용 할 때 쓰입니다.

이들 값은 특별히 접미어나 접두어를 두지 않습니다.

대신 해당 상수의 목적을 명확히 알릴 수만 있게 작성 합니다.

예시는 아래와 같습니다.

```ts
const FORM_INVALID_MESSAGE = '잘못된 입력 입니다.';

const FILE_MODIFY_SUCCESS_CODE = 202;
```

## 복수형 상수

복수형 상수는 크게 Key-value pair 인 `Dictionary` 형태와 `Array` 형태 2가지로 나뉩니다.

### Dictionary

키값 쌍의 Object 는 접미어로 `_DICTIONARY`를 붙여줍니다.

아래는 예시 입니다.

```ts
// 특정 키값이 단순 문자열일 때
const KEY_TO_NATIONAL_NAME_DICTIONARY = {
  korea: '한국',
  china: '중국',
  japan: '일본',
};

// 키값이 숫자형일 때
const SEQ_TO_ALERT_TEXT_DICTIONARY = {
  1: '잘못된 요청 입니다',
  2: '서버가 응답하지 않습니다.',
  3: '잘못된 입력 입니다.',
  4: '입력 시간을 초과 하였습니다.',
};

// 키값이 열거형(enum)일 때
const KEY_TO_SETTINGS_DICTIONARY = {
  [CategoryEnum.CLOTH]: {
    value: 123,
    name: '김서울',
    timeout: 45000,
  },
  [CategoryEnum.SHOES]: {
    value: 456,
    name: '손탠디',
    timeout: 56700,
  },
};
```

### Array

배열 형태는 접미어로 `_LIST`를 붙여줍니다.

```ts
// 단순 문자열 배열
const STEP_LABEL_FOR_SIGNUP_LIST = [
  'Step1: 아이디 적기',
  'Step2: 비밀번호 적기',
  'Step3: 이메일 체크',
  'Step4: 완료',
];

// 객체 배열
const DROPDOWN_OPTIONS_FOR_SHOES_TYPE_LIST = [
  {
    name: '구두',
    value: '567',
  },
  {
    name: '운동화',
    value: '9900',
  },
  {
    name: '샌들',
    value: '4409',
  },
];
```

## 열거형 (enum)

열거형은 기본적으로 `Pascal-case` 로 작성하되 접미어로 `Enum` 을 명시 해 줍니다.

그리고 내부 멤버는 모두 `Upper-case` 로 작성합니다.

<!-- 열거형의 선언문은 `enum`이 아닌 `const` 로 사용합니다.

(사유는 아래에 별도 설명 합니다.)-->

```ts
// 임의의 문자열을 넣을 경우
enum BuildingCategoryEnum {
  APPARTMENT = '아파트',
  ONE_ROOM = '원룸',
  TWO_ROOM = '투룸',
  OFFICTEL = '오피스텔',
  NORMAL = '일반 주택',
}

// 임의의 숫자를 넣을 경우
enum GalaxyTypeEnum {
  SPIRAL = 45,
  LENTICAL = 56,
  ELLIPTICAL = 90,
  IRREGULAR = 34,
}

// 그냥 순차적인 값을 이용할 때
enum GameGenreEnum {
  ACTION = 0,
  ROLL_PLAYING = 1,
  ADVENTURE = 2,
  STRATEGY = 3,
  SIMULATION = 4,
}
```

### 주의사항

열거형은 `연관된 상수들의 모임`으로 정의되므로, 이들 멤버들의 값은 `원시형 자료(primitive)`만을 이용하여 구성합니다.

만약 객체로 구성된 열거형이 필요하다면, 그 필요한 상황을 가능한한 지양토록 합니다.

부득이하게 그런 상황이 필요하다면 상기 언급된 `Dictionary` 형태의 상수를 사용합니다.

<!--
### const enum 사용 이유

TypeScript 의 enum 은 본래 JavaScript 에서 지원하지 않는 기능 입니다.

따라서 이를 실제 번역(Transpile)할 시 아래와 같이 IIFE(Immediately Invoked Function Expression - 즉시실행함수표현)으로 바꿔 버립니다.

```ts
// 번역전
enum ComputerEnum {
  PERSONAL,
  LAPTOP,
  NOTEBOOK,
}

const value = ComputerEnum.LAPTOP;
```

```js
// 번역후
var ComputerEnum;
(function (ComputerEnum) {
  ComputerEnum[(ComputerEnum['PERSONAL'] = 0)] = 'PERSONAL';
  ComputerEnum[(ComputerEnum['LAPTOP'] = 1)] = 'LAPTOP';
  ComputerEnum[(ComputerEnum['NOTEBOOK'] = 2)] = 'NOTEBOOK';
})((ComputerEnum = exports.ComputerEnum || (exports.ComputerEnum = {})));

const value = ComputerEnum.LAPTOP;
```

이러면 Tree-Shaking 대상에서 제외되며 만약 저 enum이 쓰이질 않을 경우 코드상에서 자동으로 제거되는 이득을 보지 못하게 됩니다.

더군다나 코드도 매우 깁니다. 😞

반면 `const enum`을 쓸 경우엔 다릅니다.

```ts
// 번역전
const enum ComputerEnum {
  PERSONAL,
  LAPTOP,
  NOTEBOOK,
}

const value = ComputerEnum.LAPTOP;
```

```js
// 번역후
// enum 선언부는 아예 없어집니다.

const value = 1; // 별도로 값을 지정하지 않았으로 두번째 인덱스인 1이 지정됩니다.
```

이렇게 본래 의도인 `상수`의 역할을 하며 불필요한 코드가 없어져 번들양도 줄어드는 이점이 있습니다.

단, 이렇게 수행하려면 `tsconfig` 의 `preserveConstEnums` 가 `false` 여야 합니다.

별도 설정하지 아니하면 기본 false 이므로 현재 모든 프로젝트에 기본으로 적용된 상태라 보시면 됩니다.
-->

## 규칙의 의의

상수의 사용 목적을 분명히 하고, 상수의 특징인 `변경할 수 없고 참조만 가능한 값`임을 활용하는 것이 목적 입니다.

또 한 여러가지 표기법으로 사용되는 혼란을 방지하고 한가지 방법으로 통일 함으로써 `이 것은 상수다` 라는 것을 코드 상에서 인지할 수 있게 하는 것이 또 다른 목적 입니다.
