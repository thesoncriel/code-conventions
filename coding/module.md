# Coding Conventions: Module

본 문서는 TypeScript 를 이용한 모듈 관련 내용과 규칙에 대하여 설명합니다.

## export default 적게 쓰기

컴포넌트나 클래스, 함수등을 모듈화 하고 `export` 할 때 아래와 같이 `default` 를 쓰는 경우가 많습니다.

```ts
export default function mergeList<T1, T2>(
  list1: T1[],
  list2: T2[]
): Array<T1 | T2> {
  // codes..
}
```

이럴 땐 default 없이 export 를 사용합니다.

```ts
export function mergeList<T1, T2>(list1: T1[], list2: T2[]): Array<T1 | T2> {
  // codes..
}
```

### 예외 사항

Page Component 의 경우엔 `code spliting` 혹은 `next.js` 같은 SSR 에서의 라우트 시스템 이용 시 entry point (진입점) 역할을 하기 때문에 `export default` 를 사용하는 것이 좋습니다.

```tsx
// GowidRootPage.tsx
export default GowidRootPage: FC = () => {
  return (
    <PageContainer title="고위드 만세!">
      <ContentContainer />
    </PageContainer>
  );
};

// routes.ts
const GowidRootPage = React.lazy(() => import('./pages/GowidRootPage'));

export const rootRoutes: ModuleRouteModel[] = [
  {
    path: '/',
    component: GowidRootPage,
  }
];
```

> Next.js 이용 시 폴더 구조를 통하여 라우트 설정을 하기에 단순히 페이지 컴포넌트 파일만 옮겨 놓으면 됩니다.
>
> 단, 여기서도 `export default` 로 페이지 컴포넌트를 내보내고 있어야 합니다.

### export default 를 왜 쓰지 않나요?

사용해서 나쁠건 없으나 일반적으로 프로젝트내 기능들을 모듈화 할 시 `index.ts` 로 하위 요소를 접근하도록 만들게 됩니다.

이유는 실제 사용처에서 아래와 같이 import 구문을 짧게 가져가기 위함 입니다.

```ts
// index 를 사용하지 않을 때
import FafaGogo from './components/FafaGogo';
import FooBarWrap from './components/FooBarWrap';
import KoreaPrideList from './components/KoreaPrideList';

// index 사용 할 때
import { FafaGogo, FooBarWrap, KoreaPrideList } from './components';
```

이 때 `export default` 만으로 구성하면, index 파일 내엔 다음과 같은 코드가 반복 됩니다.

```ts
export { default as UserInfoLabel } from './components/UserInfoLabel';
export { default as ProdItemContent } from './components/ProdItemContent';
export { default as LayoutSection } from './components/LayoutSection';
export { default as ButtonGroup } from './components/ButtonGroup';
export { default as RoundedButton } from './components/RoundedButton';
export { default as CircularIndicator } from './components/CircularIndicator';
```

이 것을 단순 `export` 된 모듈로 구성 한다면 index 파일은 다음과 같이 코드 내용이 줄어듭니다.

```ts
export * from './components/UserInfoLabel';
export * from './components/ProdItemContent';
export * from './components/LayoutSection';
export * from './components/ButtonGroup';
export * from './components/RoundedButton';
export * from './components/CircularIndicator';
```

### 규칙의 의의

특정 모듈의 `index.ts` 파일 내 코드를 짧게 가져가고 export 방법이 혼재되어 있는 상황을 통일 시킵니다.

그리고 index 파일 내 불필요한 코드량이 줄어들기에 상대적으로 코드 가독성이 좋습니다.

## 공개된 모듈은 index 로 접근하기

프로젝트를 진행 하다보면 여러가지 파일과 모듈이 만들어집니다.

이 때 아래와 같이 일반적인 `export` 와 `import`를 사용할 경우 다음과 같이 코드 내용이 많아집니다.

```ts
import { SectionWrap } from '../components/currentSection/SectionWrap';
import { RondedButton } from '../components/common/RoundedButton';
import { PageLayout } from '../components/layout/PageLayout';
import { CustomerInfoList } from '../components/list/CustomerInfoList';

// 이하 생략
```

이럴 때 공개된 모듈은 `index` 를 이용하여 `export` 해 두면 이러한 구문을 줄일 수 있습니다.

```ts
// /components/index.ts
export * from '/currentSection/SectionWrap';
export * from '/common/RoundedButton';
export * from '/layout/PageLayout';
export * from '/list/CustomerInfoList';

// 사용처
import {
  SectionWrap,
  RondedButton,
  PageLayout,
  CustomerInfoList,
} from '../components';

// 이하 생략
```

### index 는 3일 때는 필수

export 될 모듈이 같은 패키지 내 `3개`가 된다면 **필수**로 index 로 묶습니다.

가령 아래와 같은 경우 입니다.

```
.
└── modules
    ├── components
    │   ├── UserNameInput.tsx
    │   ├── AddressAutoComplete.tsx
    │   └── UserPasswordInput.tsx
    └── utils
        ├── helpers.ts
        └── collections.ts
```

이때는 아래와 같이 묶어 줍니다.

```
.
└── modules
    ├── components
    │   ├── UserNameInput.tsx
    │   ├── AddressAutoComplete.tsx
    │   ├── UserPasswordInput.tsx
    │   └── index.ts
    └── utils
        ├── helpers.ts
        └── collections.ts
```

### 1~2개일 때는 선택사항

아래 예시의 utils 처럼 1~2개 파일만 있을 경우엔 반드시 index 를 두지 않아도 됩니다. (선택사항)

```
.
└── modules
    └── utils
        ├── helpers.ts
        └── collections.ts
```

다만 1개가 늘어 3개가 되면, 이 때는 반드시 index 를 두어야 합니다.

```
// before
.
└── modules
    └── utils
        ├── helpers.util.ts
        ├── collections.util.ts
        └── etc.util.ts
```

```
// after
.
└── modules
    └── utils
        ├── helpers.util.ts
        ├── collections.util.ts
        ├── etc.util.ts
        └── index.ts
```

### export default 가 적용된 곳은 무시

default 로 export 되는 대표적인 곳이 `page component` 입니다.

이들은 각자 개별적인 import 가 이루어져야 하므로 index 를 두지 않아도 됩니다.

### Circular Dependency 주의

index 로 모듈 인터페이스를 구성하면 사용처에선 편리한점이 많으나 같은 모듈 내에서 이걸 사용 시 `순환 종속(Circular Dependency)` 문제가 발생될 수 있습니다.

프로젝트 내 아래와 같은 파일로 구성되어 있다 가정 하겠습니다.

```
.
└── modules
    ├── components
    │   ├── UserNameInput.tsx
    │   ├── AddressAutoComplete.tsx
    │   ├── UserPasswordInput.tsx
    │   ├── UserInfoSection.tsx
    │   └── index.ts
    └── pages
        └── UserPage.tsx
```

`components/index.ts` 는 다음과 같습니다.

```ts
export * from './UserNameInput.tsx';
export * from './AddressAutoComplete.tsx';
export * from './UserPasswordInput.tsx';
export * from './UserInfoSection.tsx';
```

이 중 `UserInfoSection.tsx` 가 나머지 3개를 참조 하고 있습니다.

```tsx
import {
  UserNameInput.tsx,
  AddressAutoComplete.tsx,
  UserPasswordInput.tsx,
} from './index';

export const UserInfoSection: FC = () => {
  // 자세한 코드 생략..
  return (
    <StyledWrap>
      <UserNameInput />
      <AddressAutoComplete />
      <UserPasswordInput />
    </StyledWrap>
  );
};
```

이 때 `UserPage` 에서 다음과 같이 참조 합니다.

```tsx
import { UserInfoSection } from '../components';

export default UserPage: FC = () => {
  return (
    <UserInfoSection />
  );
};
```

어떤일이 벌어질까요? 분명 `UserInfoSection.tsx` 파일에서 참조되는 `index` 때문에 `Circular Dependency` 오류가 발생 될겁니다.

이렇게 `UserInfoSection.tsx` 처럼 같은 모듈 내 다른 하위 모듈을 참조 할 때는 index 를 쓰지 말고 각각의 파일을 직접 import 합니다.

```tsx
import { UserNameInput } from './UserNameInput.tsx';
import { AddressAutoComplete } from './AddressAutoComplete.tsx';
import { UserPasswordInput } from './UserPasswordInput.tsx';

export const UserInfoSection: FC = () => {
  // 자세한 코드 생략..
  return (
    <StyledWrap>
      <UserNameInput />
      <AddressAutoComplete />
      <UserPasswordInput />
    </StyledWrap>
  );
};
```

### 규칙의 의의

모듈내 여러 객체/함수에 접근하는 코드에 일관성을 높일 수 있습니다.

또 한 다수의 import 구문을 하나의 index 로 접근을 제한 함으로써 코드의 길이도 상대적으로 짧아지게 됩니다.
