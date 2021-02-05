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
