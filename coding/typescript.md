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