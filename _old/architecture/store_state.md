# Architecture: Store State

Redux 스토어의 상태 운용 시 필요한 사전 지식과 규칙을 다룹니다.

## 정의

State 는 Store 의 구성요소 중 하나로서 단어의 의미 그대로 UI의 `상태`를 가집니다.

상태는 크게 3가지로 나뉩니다.

1. Local State - 특정 UI Component 만 가지고 그 범위(scope)에서만 쓰임.
2. Feature State - 사용되는 페이지나 모듈 범위에서 쓰임.
3. Shared State - 프로젝트 전반에 걸쳐 공유하며 사용됨.

이 중 Flux Architecture 에서 주로 이용되는 방식인 2,3 번에 대해 먼저 알아 보겠습니다.

## 운용 패턴

### Presentation Model

표현 모델은 오로지 View UI 에 특화된 전용 DTO 입니다.

일반적으로 디자인 시안이나 기획서의 스토리 보드를 바탕으로 만들어지며 Backend 의 Entity 와는 무관합니다. (물론 필드에 따라 유사성은 가질 수 있습니다.)

따라서 Repository 를 통해 받아온 Entity 는 `Converter` 를 통해 `UI Model` 로 변환되어 쓰입니다.

그리고 State 는 바로 이 `UI Model` 로만 구성됩니다.

### Factory Method

State 는 일반적으로 Plain Object 로 구성되며 이들을 초기화 하는 것은 일반적으로 Reducer 나 Slice 의 `getInitState` 함수가 맡습니다.

이들은 일종의 `Factory` 메서드로 취급되며 로직 구성 중 하위 객체들은 모두 `creator` 가 맡게 됩니다.

> Creator (또는 Create) 는 Store 구성 요소 중 하나로서, UI Model 이나 DTO 를 같은 내용으로 생성해 주는 함수들을 의미 합니다.

특별한 사유가 없는 한 UI Model 은 초기화엔 `creator` 가, 변환은 `converter` 에서만 이루어져야 합니다.

### Null Object

State 는 특수한 경우를 제외하고는 null 이나 undefined 등의 `Nullish` 를 허용하지 않습니다.

즉, 각 필드들은 Optional 이 되어서는 안됩니다.

이유는 다음과 같습니다.

1. 데이터 로직 중 지속적으로 타입 체킹을 해야 합니다.
2. 프로그래머의 실수로 오류가 생기기 쉽습니다.
3. 상기 이유로 운용 시 안정성이 떨어집니다.

따라서 null 이 필요하다면, null 을 대체하는 객체를 대신 넣습니다.

이렇게 nullable, optional 을 허용하지 않고 그 필드에 호환되는 기본 객체를 넣어주는 것을 통틀어 `Null Object` 패턴이라 부릅니다.

아래는 통상적으로 개발되는 방식의 예제 입니다.

```ts
interface FeatState {
  info?: UserInfoModel; // optional 이기 때문에 nullable 입니다.
  loading: boolean;
}

const effFetchUserInfo = createAsyncThunk<UserInfoModel | undefined, void>(
  async () => {
    const res = await repo.user.fetchInfo();

    // 응답이 null 일 경우 info 는 undefined 가 됩니다.
    const info = toUserInfoModel(res);

    return info;
  }
);
```

```tsx
interface UserInfoViewProps {
  info?: UserInfoModel;
}

const UserInfoView: FC<UserInfoViewProps> = ({ info }) => {
  // info 의 존재 유무에 따라 체킹하는 코드의 양이 크게 늘어납니다.
  const welcome = useMemo(() => {
    if (info && info.welcome && info.welcome.kits && info.welcome.kits.length > 0) {
      return info.welcome.kits.join(',');
    }
    return '';
  }, [info]);


  return (
    <>
      <h2>환영합니다~!</h2>
      <dl>
        <dt>이름 : <dt>
        <dd>{info ? info?.name : '-'}
        <dt>나이 : <dt>
        <dd>{info?.age || '알 수 없음'}</dd>
      </dl>
      <blockquote>
        제공된 키트: {welcome}
      </blockquote>
    </>
  );
};
```

아래는 위 코드를 `Null Object Patern` 으로 적용 한 코드 입니다.

```ts
interface FeatState {
  info: UserInfoModel; // 이제 사용자 정보는 필수입니다.
  loading: boolean;
}

function toUserInfoModel(entity: UserInfoEntity | null): UserInfoModel {
  if (!entity) {
    return createUserInfoModel();
  }
  return {
    name: entity.name || '',
    age: entity.age || 0,
    welcome: {
      kits: getWelcomeKitsFrom(entity),
      kitsDetail:
    }
  }
}

const effFetchUserInfo = createAsyncThunk<UserInfoModel, void>(
  async () => {
    const res = await repo.user.fetchInfo();

    // 응답이 null 일 경우 createUserInfoModel 를 자체적으로 실행 합니다.
    // 따라서 결과는 절대 undefined 가 될 수 없습니다.
    const info = toUserInfoModel(res);

    return info;
  }
);
```

```tsx
interface UserInfoViewProps {
  info: UserInfoModel; // 이젠 필수 입니다.
}

const UserInfoView: FC<UserInfoViewProps> = ({ info }) => {
  // info 와 하위 데이터의 존재를 신뢰할 수 있으므로 그저 가져다 쓰면 됩니다!
  const welcome = useMemo(() => {
    return info.welcome.kits.join(',');
  }, [info]);

  return (
    <>
      <h2>환영합니다~!</h2>
      <dl>
        <dt>이름 : <dt>
        <dd>{info.name : '-'}
        <dt>나이 : <dt>
        <dd>{info.age || '알 수 없음'}</dd>
      </dl>
      <blockquote>
        제공된 키트: {welcome}
      </blockquote>
    </>
  );
};
```

### optional enum 및 멤버 초기화

> 아래 규칙은 BE API 에서 해당 필드의 값이 optional 이 아니라면 굳이 해 줄 필요는 없습니다. 😅

분명 BE API 의 Entity 로는 지정된 값이 들어오지만, 아직 처리되기전 이라던가, 정말 값이 없어서 null 로 오는 경우가 있습니다.

이 때 단순 문자열이라면 빈 값(empty)을 주시면 됩니다.

```ts
function toTemporaryProductionUIModel(entity: ProductionEntity) {
  return {
    name: entity.detail.name || '',
    // 이하 생략..
  }
}
```

만약 숫자형이라면 0(zero)를 주시면 됩니다.

```ts
function toSafeMoneyUIModel(entity: SafeMoneyEntity) {
  return {
    price: entity.cost || 0,
    // 이하 생략..
  }
}
```

한편 enum 같은 경우, 보통 기본값에 대응되는 것들이 눈에 띄게 마련 입니다.

> 초기화에 대응되는 enum 키워드로는 `default` 나 `initialize`, `none`, `unknown`, `first`, `1step`, `empty` 등이 있습니다.
>
> 꼭 위에 제시된 키워드가 아니더라도 업무 문맥상, 초기화에 대응되는 것이라면 그것을 사용 하시면 됩니다.

다만, 정말 초기화에 대응되는 것이 없을 수 있습니다.

이 때 enum 같은 경우엔 `NONE` 이나 `UNKNOWN` 같은 걸 골라서 아래와 같이 미리 설정 한뒤 그 값을 초기 값으로 대신 사용 합니다.

```ts
// before
function toWorkingProcessUIModel(entity: WorkingProcessEntity) {
  return {
    name: entity.name,
    type: entity.type
      ? (entity.type as WorkingProcessEnum)
      : null
  };
}

// after
function toWorkingProcessUIModel(entity: WorkingProcessEntity) {
  return {
    name: entity.name,
    type: entity.type
      ? (entity.type as WorkingProcessEnum)
      : WorkingProcessEnum.NONE,
  };
}
```

문자열 enum 예시

```ts
// 원본
enum WorkingProcessEnum {
  WORKING = 'WORKING',
  REVIEW = 'REVIEW',
  DONE = 'DONE'
}
// 추가
enum WorkingProcessEnum {
  NONE = '', // 빈 값을 기본으로 줍니다.
  WORKING = 'WORKING',
  REVIEW = 'REVIEW',
  DONE = 'DONE'
}
```

숫자형 enum 예시

```ts
// 원본
enum WorkingProcessEnum {
  WORKING = 100,
  REVIEW = 200,
  DONE = 300
}
// 추가
enum WorkingProcessEnum {
  NONE = 0, // 0 값을 기본으로 줍니다.
  WORKING = 100,
  REVIEW = 200,
  DONE = 300
}
```

만약 0 값이 이미 API 에 명시되어 있다면, -1 을 대신 사용 합니다.

```ts
// 원본
enum WorkingProcessEnum {
  WAITING = 0,
  WORKING = 100,
  REVIEW = 200,
  DONE = 300
}
// 추가
enum WorkingProcessEnum {
  UNKNOWN = -1, // -1 값을 기본으로 줍니다.
  WAITING = 0,
  WORKING = 100,
  REVIEW = 200,
  DONE = 300
}
```

참고로 none 및 unknown 은 임의로 선택 해 둔 것입니다.

#### 주석으로 명시

초기화에 대응되는 enum 필드를 프론트에서 임의로 만들었을 경우, 업무의 혼선을 방지하기 위해 다음과 같이 주석을 추가하고 하단에 관련 내용을 명시 해 둡니다.

```ts
enum WorkingProcessEnum {
  /**
   * 해당사항 없음
   * 
   * - made by FE
   */
  NONE = 0,
  /**
   * 열일 하는 중
   */
  WORKING = 100,
  /**
   * 리뷰 진행중
   */
  REVIEW = 200,
  /**
   * 업무 종료
   */
  DONE = 300
}
```

보통 Entity 출신의 enum 은 BE API 의 것을 그대로 활용하게 될텐데, 이렇게 명시 해 두면 작업자가 임의로 추가 해 둔 것인지를 알 수 있습니다. 👍

### 타입 일관성

State 의 멤버(필드)들은 가능한 한 1가지의 타입만 선언 합니다.

```ts
// wrong
// 하나의 필드에 여러가지 타입을 두지 않습니다.
// 이러면 데이터 사용처에서 매번 타입 체킹을 해야 합니다.
interface FeatState {
  age: number | string | null;
  address:
    | string
    | {
        detail: string;
        zipcode: string;
      };
}

// good
interface FeatState {
  // 빈 값 (empty)이 공존해야 한다면 string 으로 통일 시킵니다.
  age: string;
  // 결국 원하는 게 주소값이라면 converter 나 creator 에서 충분히 걸러서 주도록 합니다.
  address: string;
}
```

### 패턴 적용의 이점

백엔드 엔티티와 API 변화에도 무관한 독립적인 View UI 가 만들어 집니다.

Null 을 허용하지 않는 데이터 구조는 하위 로직이나 컴포넌트에서 타입 체킹을 최소화 하여 믿고 쓸 수 있는 기반이 됩니다.

또 한 타입의 일관성을 유지하여 불필요한 코드를 미연에 방지할 수 있습니다.

즉 이 모든 사항들은 `외부 환경과의 관심사 분리`와 `개발 오류 최소화`를 위한 것 입니다.
