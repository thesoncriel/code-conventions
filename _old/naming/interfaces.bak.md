# Naming Conventions: Interfaces

TypeScript 의 인터페이스는 다음과 같이 크게 2가지 쓰임새를 가질 수 있습니다.

1. VO, DTO 나 struct 등 데이터 교환 목적
2. 추상화를 위해 클래스(or 객체)가 가지는 기능들만 선언 해 놓은 껍데기.

인터페이스 명칭은 반드시 `Pascal notation` 으로만 작성 합니다.

## 데이터 구조의 목적

공통적으로 `접미어(postfix)`를 가집니다.

이들은 크게 2가지 목적으로 구분할 수 있으며 일반적으로 사용자 계층인 `Client` 와 제공자 계층인 `Server` 로 나뉩니다.

### Client Side

클라이언트는 오로지 View 와 그 부속 기능인 Store 및 내부 Service 등 `서버와는 관계없는` 자료를 다루는 영역 입니다.

- Presentation Model
  - 표현 모델. 오로지 클라이언트에서만 쓰이는 자료
- Store, Component State
  - 스토어나 컴포넌트가 가질 수 있는 상태. 일반적으로 표현 모델을 조합하거나 가지고 있다.
- Component Props
  - 컴포넌트 속성들. 일반적으로 표현 모델을 기반으로 구성된다.
- Event Arguments
  - 컴포넌트에서 발생된 각종 이벤트 발생 시 전달되는 자료들.
- Action Payload
  - 액션이 수행될 때 함께 보내어지는 자료. FSA 기준, type 을 제외한 나머지를 의미한다.
- Page Query Parameter
  - 웹브라우저에서 페이지 호출 시 입력되는 각종 쿼리 파리미터를 객체화 시켜 놓은 것.

| 명칭                   | 접미어  | 예시                                     |
| :--------------------- | :------ | :--------------------------------------- |
| Presentation Model     | PM      | UserInfoPM, BuildingDataPM               |
| Store, Component State | State   | BoardListState, UserSignUpState          |
| Component Props        | Props   | InputGroupListProps, ProductItemProps    |
| Event Arguments        | Args    | SNSItemClickArgs, PasswordChangeArgs     |
| Action Payload         | Payload | NotificationPayload, FeedbackStopPayload |
| Page Query Parameter   | Queries | RootPageQueries, BoardPageQueries        |

#### 작성 예시

아래는 인스타그램과 유사한, 특정 사용자의 정보를 바탕으로 꾸며진 웹앱을 가정 하였습니다.

그 중 인터페이스 선언부만 추려서 간략하게 구성 해 본 예시 입니다.

세부적인 코드는 이 후 만들어진 규칙과 다소 차이가 있을 수 있으므로 사용처와 네이밍만 참고 하시기 바랍니다. 😄

또 한 아래 내용들은 서로 연관되어 하나의 완전한 프로그램이 되는 코드가 아니므로 이 또한 참고를 부탁드립니다.

```ts
// Presentation Model
interface UserInfoPM {
  id: number;
  name: string;
  age: number;
  pictureUrl: string;
  likeCount: number;
  dislikeCount: number;
  postCount: number;
}
```

```ts
// Store State
interface TodayUserState {
  items: UserInfoPM[];
  loading: boolean;
}

// ...with reducer
export function reducer(state: TodayUserState, action: TodayUserAction) {
  switch (action.type) {
    case TodayUserActionEnum.LOAD:
      return {
        ...state,
        loading: true,
      };
    case TodayUserActionEnum.SUCC:
      return {
        items: action.payload,
        loading: false,
      };

    default:
      return state;
  }
}
```

```ts
// Component Props
// 현재 컴포넌트에서 쓰고 말 내용이면 그냥 'Props'로 간단히 정의 합니다.
// 현재 컴포넌트는 표현 모델을 바탕으로 구성되기에 설계상 이미 존재하는 interface 를 상속받아 작성 됩니다.
interface Props extends UserInfoPM {
  // 표현 모델 기반에 이벤트가 추가 되었습니다.
  // 나머지 필드는 모두 표현 모델의 것을 그대로 상속 받습니다.
  onSnsSelect?: (sns: SNSEnum) => void;
}

export const UserInfoPanel: FC<Props> = ({ onSnsSelect, ...props }) => {
  // 생략
};
```

```ts
// Component Props
// 만약 표현 모델 그대로 사용 한다면 아래와 같이 표현 모델을 그대로 쓰거나 type 으로 alias 하여 사용합니다.

import { UserInfoPM } from '../dto';

export const UserInfoPanel: FC<UserInfoPM> = ({ onSnsSelect, ...props }) => {
  // 생략
};

// ---- or ----

import { UserInfoPM } from '../dto';

type Props = UserInfoPM;

export const UserInfoPanel: FC<Props> = ({ onSnsSelect, ...props }) => {
  // 생략
};
```

```ts
// Component Props
// 만약 같은 props 를 가진 컴포넌트가 다수 있어 Props 를 외부에 공개할 때는
// 이와 같이 컴포넌트 풀네임에 Props 를 붙여 줍니다.
export interface BaseNotificationProps {
  // 생략
}

export const BaseNotification: FC<BaseNotificationProps> = (props) => {
  // 생략
};
```

```tsx
// Event Arguments
// 이벤트 아규먼트는 이벤트 발생 시 함께 전달하고픈 자료가 여럿 묶여있을 때 사용한다.
interface UserFollowArgs {
  userId: number;
  sns: SNSEnum;
}

// --- 이하 사용되는 컴포넌트 ---

import { UserFollowArgs } from '../dto';

interface Props {
  user: UserInfoPM;
  sns: SNSEnum
  onFollow?: (args: UserFollowArgs) => void;
}

export const UserFollowablePostItem: FC<Props> = ({ onFollow, sns, user }) => {
  const handleClick = () => {
    onFollow && onFollow({ sns, userId: user.id });
  };

  return (
    <div>
      <h3>팔로우 하기</h3>
      <p>블라블라블라..</p>
      <button type="button" onClick={handleClick}>
    </div>
  );
}
```

```tsx
// Action Payload
// 특정 사용자가 팔로우 할 때 그 액션 수행과 함께 넘겨줄 자료를 정의한 모습.
interface UserFollowPayload {
  userId: number;
  sns: SNSEnum;
  time: number;
}

// 만약 페이로드 구조가 이벤트 아규먼트와 구조가 유사하다면 아래와 같이 인터페이스 상속하여 사용해도 좋습니다.
interface UserFollowPayload extends UserFollowArgs {
  time: number;
}

// --- 액션 선언부 ---
export const actUserFollow = (payload: UserFollowPayload) => ({
  type: ACTION_FOLLOW,
  payload,
});

// --- 이하 사용되는 컨테이너 ---
import { UserFollowPayload, UserFollowArgs } from '../dto';

interface Props {
  userId: number;
}

export function UserFollowPanelContainer() {
  const user = useSelector((items) => items[0]);
  const dispatch = useDispatch();

  const handleFollow = (args: UserFollowArgs) => {
    dispatch(
      actUserFollow({
        userId: args.userId,
        sns: args.sns,
        time: Date.now(),
      })
    );
  };

  return (
    <UserFollowablePostItem
      user={user}
      sns={SNSEnum.FACEBOOK}
      onFollow={handleFollow}
    />
  );
}
```

```tsx
// Page Query Parameter
// 페이지에서 쓰이는 쿼리 파라미터. 인터페이스 선언은 대체로 해당 페이지 내에서 이뤄진다.

interface UserInfoPageQueries {
  id: string;
  name: string;
}

export default UserInfoPage: FC = () => {
  const query = useQueryParams<UserInfoPageQueries>();

  return (
    <PageContainer title="사용자 정보">
      <UserListPanelContainer id={query.id} name={query.name} />
    </PageContainer>
  );
};
```

### Server Side

서버는 Backend API 를 통해 주고 받는, `UI와는 관계없는` 자료를 다루는 영역 입니다.

이들은 보통 `Repository` 계층에서 사용 됩니다.

- Entity
  - 서버에서 전달 해 준 자료. 응답 자료에 단일로 구성 되거나 **Response Data**의 하위 자료로써 구성될 수 있다.
- Request Parameter
  - 서버에 자료 요청 시 필요한 자료. get/delete 수행 시 사용되는 쿼리 파라미터 뿐만 아니라 put/post 에서 사용되는 request body 도 함께 포함하고 있다.
  - 즉 레포지토리 계층을 사용할 때 이들이 실제 query parameter 인지, request body 인지에 대해서는 관심을 가지지 않도록 만든다.
- Response Data
  - 응답 데이터. 일반적으로 공통으로 사용되는 구조에 정의된다.
  - 따라서 API 호출 결과에 반드시 포함되진 않는다.

| 명칭              | 접미어 | 예시                                    |
| :---------------- | :----- | :-------------------------------------- |
| Entity            | Entity | AuthInfoEntity, CompanyStyleEntity      |
| Request Parameter | Params | UserSignupParams, AddingBoardItemParams |
| Response Data     | Res    | ListItemsRes, ErrorRes, StatusRes       |

#### 작성 예시

```ts
// Entity
// 백엔드에서 바로 불러온 raw data 입니다.
interface UserEntity {
  id: number;
  name: string;
  age: number;
  createdAt: string;
  updatedAt: string;
  followCount: number;
  picture: UserPicture;
}

interface UserPicture {
  path: string;
  sizeWidth: number;
  sizeHeight: number;
}

// repository 사용 예시
export function fetchUserInfo(id: number): Promise<UserEntity> {
  // 생략
}
```

```ts
// Request Parameter
// API 요청 시 쓰이는 파라미터 입니다.
interface UserListParams {
  skip: number;
  limit: number;
  keyword?: string;
}

// repository 사용 예시
export function fetchUserList(params: UserListParams): Promise<UserEntity[]> {
  // 생략
}
```

```ts
// Request Parameter
// request body 일 경우의 예시 입니다.
interface UserDetailParams {
  nickName: string;
  name: string;
  age: number;
}

// repository 사용 예시
export function sendUserInfo(params: UserDetailParams): Promise<void> {
  return axios.post({
    url: '/user',
    data: params,
  });
}
```

```ts
// Response Data
// 응답 데이터 유형이 반복되고 내부 엔티티만 변화될 때 쓰입니다.
interface ItemsRes<T> {
  items: T[];
  totalCount: number;
  skip: number;
  limit: number;
}

// repository 사용 예시
export function fetchProductList(
  params: ProductListParams
): ItemsRes<ProductItem> {
  // 생략..
}
```

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
