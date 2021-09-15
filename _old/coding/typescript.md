# Coding Conventions: TypeScript

본 문서는 타입 스크립트 코딩 시 지켜야 할 규칙에 대하여 설명합니다.

## 즉석 타입 선언 금지

타입스크립트로 제네릭 이용 시 아래와 같이 타입을 별도 선언하지 않고 즉석에서 선언하여 사용하는 경우가 있습니다.

이럴 땐 즉석 타입 선언보다 `interface`를 통한 타입 선언을 이용합니다.

### 예시1 - 레포지토리 함수 선언 사용

```tsx
// before
function fetchList(params: ListParams) {
  return baseApi.get<{ data: ResItem[]; totalCount: number }>(
    '/api/path/',
    params
  );
}
```

```ts
// after
interface ListRes {
  data: ResItem[];
  totalCount: number;
}

function fetchList(params: ListParams) {
  return baseApi.get<ListRes>('/api/path/', params);
}
```

### 예시2 - 스타일 컴포넌트 props 선언 사용

```ts
// before
const StyledListItem = styled.li<{ selected: boolean; even: boolean }>`
  border-color: ${({ selected }) => (selected ? 'red' : 'paleblue')};
  background-color: ${({ even }) => (even ? 'skyblue' : '#fff')};
`;
```

```tsx
// after
interface ListItemProps {
  selected: boolean;
  even: boolean;
}
const StyledListItem = styled.li<ListItemProps>`
  border-color: ${({ selected }) => (selected ? 'red' : 'paleblue')};
  background-color: ${({ even }) => (even ? 'skyblue' : '#fff')};
`;
```

### 예시3 - 컬렉션 수행 동작

```ts
// before
interface CupModel {
  name: string;
  material: MaterialEnum;
  price: number;
  type: CupTypeEnum;
}

function collectPrices(
  priceFieldItems: Array<{ price: number }>
): Array<{ price: number }> {
  return priceFieldItems.map((item) => ({
    price: item.price,
  }));
}

// 사용부
const cupItems: CupModel[] = getFromSomeWhere();
const result: Array<{ price: number }> = collectPrices(cupItems);
```

```ts
// after
interface PriceFieldModel {
  price: number;
}
interface CupModel extends PriceFieldModel {
  name: string;
  material: MaterialEnum;
  type: CupTypeEnum;
}

function collectPrices(priceFieldItems: PriceFieldModel[]): PriceFieldModel[] {
  return priceFieldItems.map((item) => ({
    price: item.price,
  }));
}

// 사용부
const cupItems: CupModel[] = getFromSomeWhere();
// result 타입이 PriceFieldModel[] 여도 좋습니다!
const result: Array<PriceFieldModel> = collectPrices(cupItems);
```

### 규칙의 의의

매번 즉석으로 선언하고 사용 시 타입이 파편화 되고 일관성을 흐리게 만들 수 있습니다.

반면 인터페이스로 선언하여 사용하면, 타입 선언 및 사용에 일관성 뿐만 아니라, 타입 이용의 반복적인 패턴이 있다면, 그 타입에 대한 재사용이 가능해 집니다.

이렇게 제네릭이 적용되는 코드에서는 그 타입의 세부적인 내용을 알지 않아도 되므로 상호간에 관심사 분리가 이뤄질 수 있습니다.

## DTO 선언 시 하위 DTO 는 별도로 선언하기

DTO 선언 시 하위 타입에 대한 depth 가 깊어지는 경우가 있습니다.

이 때 한 곳에 몰아서 코딩하면, 당장은 한눈에 들어와 보기 좋을 수 있습니다.

```ts
interface HouseModel {
  name: string;
  addressInfo: {
    location: {
      lng: number;
      lat: number;
    };
    address: string;
  };
  pictures: Array<{
    title: string;
    src: string;
    size: {
      width: number;
      height: number;
    }
  }>
}
```

하지만 이 세부적인 하위 자료를 사용하는 곳에서는 아래와 같이 하위 타입을 물고 가야 합니다.

```ts
// 가로 x 세로 를 하여 넓이를 구합니다.
function calcPictureArea(house: HouseModel) {
  return house.pictures[0].size.width * house.pictures[0].size.height;
}
```

이럴땐 인터페이스의 key 접근자를 이용할 수 있지만 하나의 타입으로 정하지 않았으므로 상대적으로 지저분해 보입니다.

```ts
// 가로 x 세로 를 하여 넓이를 구합니다.
function calcPictureArea(size: HouseModel['pictures'][0]['size']) {
  return size.width * size.height;
}
```

### 하위 DTO 는 별도로 타입을 선언

이렇게 타입의 depth 가 깊어지면 여러 문제로 인하여 타입의 재사용성이 떨어집니다.

이런 경우, 아래와 같이 별도의 타입을 선언하여 연결시켜 줍니다.

```ts
interface LocationModel {
  lng: number;
  lat: number;
}

interface AddressInfoModel {
  location: LocationModel;
  address: string;
}

interface PictureSizeModel {
  width: number;
  height: number;
}

interface PictureModel {
  title: string;
  src: string;
  size: PictureSizeModel;
}

interface HouseModel {
  name: string;
  addressInfo: AddressInfoModel;
  pictures: PictureModel[];
}
```

최종 DTO 의 선언부가 굉장히 깔끔해 졌습니다.

이젠 사용할 땐 미리 선언된 타입을 지정하면 됩니다.

```ts
// 가로 x 세로 를 하여 넓이를 구합니다.
function calcPictureArea(size: PictureSizeModel) {
  return size.width * size.height;
}
```

### 규칙의 의의

하나의 DTO 는 하나의 고유한 도메인을 가지고, 하위 속성과 객체 역시 고유의 도메인을 가집니다.

따라서 이들은 각자 별도의 영역에서 자료를 전달하는 역할을 하게됩니다.

즉 하위 도메인은 상위 도메인의 상세한 정보를 몰라도 되므로 자연스레 관심사 분리가 이루어 집니다.

또 한 자료를 분리하여 선언하면 이들에 대한 코드의 재사용성도 높일 수 있습니다.

만약 이들을 기반으로 또 다른 자료가 만들어진다면 자료의 확장성도 덤으로 가져갈 수 있습니다.


## setInterval 보다 setTimeout 활용하기

setInterval 은 다음과 같은 이유로 사용을 지양합니다.

- setTimeout 과는 달리 지연시간(delay time)을 보장하지 않습니다.
- 동작 중 alert 이나 confirm 이 떠도 멈추지 않습니다.
- clearInterval 을 하기 전 까진 사실상 제어가 불가 합니다.
- 지연시간보다 내부 함수 수행 시간이 길어지면 멈추지 않고 끊임없이 내부 함수를 수행하게 됩니다.

따라서 아래와 같이 setInterval 을 사용 할 일이 있다면 setTimeout 을 대신 사용합니다.

```ts
let time = 0;
function setTime(val: number) {
  time = val;
  console.log(val);
}

// wrong
const timerId = setInterval(() => {
  setTime(time + 1);

  if (time >= 10) {
    clearInterval(timerId);
  }
}, 1000);

// good~~!!!
// 재귀 호출 할 함수를 만들어 줍니다.
function doTimer() {
  setTimeout(() => {
    setTime(time + 1);

    if (time < 10) {
      doTimer();
    }
  }, 1000);
}
doTimer(); // 첫 수행이 필요합니다.
```

만약 수행중 조건에 따른 중단이 필요하다면 아래와 같은 코드를 삽입하여 제어합니다.

```ts
// 타임아웃 ID 를 받아올 변수 선언
let timeId = 0;
function doTimer() {
  // setTimeout 수행 후 ID를 받아옵니다.
  timeId = setTimeout(() => {
    setTime(time + 1);

    if (time < 10) {
      doTimer();
    }
  }, 1000);
}

// 필요 시 수행되는 timeout 을 멈추는 함수 입니다.
function stop() {
  clearTimeout(timeId);
}

document
  .getElementById('btn_stop')
  .addEventListener('click', stop);
```

### 규칙의 의의

같은 결과지만 좀 더 제어하기 쉬운 방법을 선택하여 예기치 못한 오류를 줄일 수 있습니다.

그리고 setTimeout 과 setInterval 을 혼합 사용하여 발생되는 코드 파편화를 줄입니다.

### 참고
- [중첩 setTimeout](https://ko.javascript.info/settimeout-setinterval)
- [피들러 예제](https://jsfiddle.net/4pLgcsk6/1/)


## 즉시 실행 함수(IIFE) 사용 금지

>  IIFE - Immediately Invoked Function Expression

특별한 사유가 없다면 아래와 같은 즉시 실행 함수를 작성 하지 않습니다.

```ts
// before
function calcProductAmount(args: ProductChangeArgs) {
  const amount = (() => {
    const {
      price,
      quantity,
    } = args;
    
    return price * quantity;
  })();

  const taxValue = (function(amt: number) {
    if (amt > 0) {
      return 0;
    }

    return amt * 0.1;
  })(amount);

  return {
    ...args,
    amount,
    taxValue,
  };
}
```

위와 같은 내용은 다음과 같이 작성 합니다.

```ts
// after
function calcAmount(price: number, quantity: number) {
  return price * quantity;
}

function calcTax(amt: number) {
  if (amt > 0) {
    return 0;
  }

  return amt * 0.1;
}

function calcProductAmount(args: ProductChangeArgs) {
  const amount = calcAmount(args.price, args.quantity);
  const taxValue = calcTax(amount);

  return {
    ...args,
    amount,
    taxValue,
  };
}
```

> 상기 예제는 지면상 짧은 내용을 사용 하였습니다.
>
> 실제 업무 적용 시 저렇게 과도하게 짧은 함수를 여럿 만들 필요는 없습니다. 😅

### 이런게 왜 있는건가요?

본디 JS는 global 과 local scope, 2개의 영역밖에 존재하지 않았습니다.

그래서 IIFE 가 고안되었고 다음과 같은 상황에 쓰였습니다.

- module pattern
  - 쓰이는 코드 내용들을 특정 영역(scope)에 가둬둠으로써 global 기능과 분리하고 격리하기 위함입니다.
- 보안성 강화
  - 정해진 스코프 내의 변수/함수는 외부로 return 하지 않는 한 절대 접근이 불가 합니다.

요약하자면 보안성 강화 & 영역 분리가 있겠습니다.

### 지금은 왜 사용하면 안되나요?

ES6가 나오면서 `block scope` 가 생겼습니다.

그래서 특정 함수 내 별도의 영역 구분이 필요하다면 굳이 IIFE를 쓰지 않아도 됩니다.

그리고 IIFE는 webpack 으로 번들링 할 때 tree-shaking 대상이 되지 않거나 예기치 못한 side-effect 가 발생될 수 있습니다.

그리고 단순히 연산의 영역을 구분하기 위함이라면 `별도로 함수를 만들어 쓰기`를 권장 합니다.

왜냐하면 IIFE 사용시, 매번 함수 수행 될 때마다 재생성 & 실행 되기 때문에 이미 만들어진 함수를 사용하는 것 보다 상대적으로 overhead가 크기 때문입니다.

### 규칙의 의의

내부 로직을 별도 함수로 구성함으로써 자연스레 코드 분리가 됩니다.

그리고 필요시 같은 내용에 대한 `재사용성 증가`가 이뤄질 수 있습니다.

내부 함수의 오버헤드를 줄이고 간결한 코드를 유지할 수 있습니다.

### 참고

- [Webpack Tree shaking](https://programmersought.com/article/29911737900/)