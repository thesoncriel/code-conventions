# Naming Conventions: Interfaces - Client Side

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
- Data Transfer Object - DTO
  - 상기 언급된 그 어디에도 속하지 않는 용도의 데이터 구조.
  - 예: 함수대 함수, 혹은 클래스 끼리 주고받는 내용들

| 명칭                   | 접미어  | 예시                                     |
| :--------------------- | :------ | :--------------------------------------- |
| Presentation Model     | Model   | UserInfoModel, BuildingDataModel         |
| Store, Component State | State   | BoardListState, UserSignUpState          |
| Component Props        | Props   | InputGroupListProps, ProductItemProps    |
| Event Arguments        | Args    | SNSItemClickArgs, PasswordChangeArgs     |
| Action Payload         | Payload | NotificationPayload, FeedbackStopPayload |
| Page Query Parameter   | Queries | RootPageQueries, BoardPageQueries        |
| Data Transfer Object   | DTO     | GoodsQuantityCalcDTD, ImageProcecingDTO  |

## 작성 예시

아래는 인스타그램과 유사한, 특정 사용자의 정보를 바탕으로 꾸며진 웹앱을 가정 하였습니다.

그 중 인터페이스 선언부만 추려서 간략하게 구성 해 본 예시 입니다.

세부적인 코드는 이 후 만들어진 규칙과 다소 차이가 있을 수 있으므로 사용처와 네이밍만 참고 하시기 바랍니다. 😄

또 한 아래 내용들은 서로 연관되어 하나의 완전한 프로그램이 되는 코드가 아니므로 이 또한 참고를 부탁드립니다.

```ts
// Presentation Model
interface UserInfoModel {
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
  items: UserInfoModel[];
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
interface Props extends UserInfoModel {
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

import { UserInfoModel } from '../dto';

export const UserInfoPanel: FC<UserInfoModel> = ({ onSnsSelect, ...props }) => {
  // 생략
};

// ---- or ----

import { UserInfoModel } from '../dto';

type Props = UserInfoModel;

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
  user: UserInfoModel;
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
