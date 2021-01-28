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
