# Architecture: Presentational Component

본 문서는 아키텍처 운용 방법 중 `표현 컴포넌트` 에 대하여 설명 합나다.

## 정의

표현 컴포넌트란 순수함수(pure function)와 그 맥락이 거의 같습니다.

즉 오로지 `전달 받은 props 를 바탕으로 표현`만 해야하며, 전달 받은 props 의 event 들을 통해 사용자의 행위들을 trigger 하는 것이 주요 목적 입니다.

> 표현 컴포넌트의 핵심은
>
> story book 같은 visual test 환경에서 테스트가 가능하도록 만드는 것입니다.
>
> 이를 통해 컴포넌트의 캡슐화(encapsulation)와 관심사분리(separation of concerns)를 유도하여 느슨한 결합(loose coupling)이 되도록 만듭니다.

## 특징

표현 컴포넌트는 아래와 같은 특징을 가집니다.

1. 어디서든 제약없이 이용 가능하다.
2. 이들의 의존성(dependency)은 디자인 시스템 기반, 혹은 다른 표현 컴포넌트만을 바라봐야 한다.
3. 주어지는 props 외 바깥 환경이나 데이터에 영향을 받아서는 안된다.

## 제약

이들을 운용하기 위해선 다음과 같은 규칙을 가져야 합니다.

1. Flux Architecture 기준, Store 의 영역을 직접적으로 이용하지 않는다.
2. 복잡한 데이터 조작 로직을 가지지 않는다.
3. 사용되는 hooks 나 utility 는 오직 visual logic 만으로 구성 되어야 한다.

### Flux Architecture 기준, Store 의 영역을 직접적으로 이용하지 않는다.

표현 컴포넌트는 store 를 대상으로 하는 dispatch 나 action 등을 할 수 없습니다.

```tsx
// wrong
export const UserSection: FC = () => {
  const dispatch = useDispatch();
  const { name, age } = useSelector(selUserState);

  const handleClick = () => {
    dispatch(effUserSubmit());
  };

  return (
    <section>
      name: {name}<br />
      age: {age}<br />
      <button type="button" onClick={handleClick}>submit</button>
    </section>
  );
};
```

```tsx
// good
interface UserSectionProps {
  name: string;
  age: number;
  onSubmit: () => void;
}

export const UserSection: FC<UserSectionProps> = ({
  name,
  age,
  onSubmit
}) => {
  return (
    <section>
      name: {name}<br />
      age: {age}
      <button type="button" onClick={onSubmit}>submit</button>
    </section>
  );
};
```

또 한 외부 데이터를 직접적으로 가져오는 행위를 할 수 없습니다.

```tsx
// wrong
export const UserSection: FC = () => {
  // 라우트 파라미터를 가져오는 것은 최소 Container 이상이어야 합니다.
  const params = useParams();
  // useState 를 사용할 순 있으나 그 목적이 외부 데이터를 호출하고 보관하는 용도여선 안됩니다.
  const [state, setState] = useState(null);
  const [count, setCount] = useState(0);
  const [workings, setWorkings] = useState([]);

  useEffect(() => {
    // 표현 컴포넌트에선 레포지토리나 fetch, axios 등 XHR 계통 명령을 수행하면 안됩니다.
    repo.user.userInfo().then(res => {
      setState(res);
    });
  }, []);

  useEffect(() => {
    // 쿠키나 로컬스토리지 사용은 허용되지 않습니다.
    const countValue = Number(cookie.get('count'));
    const workingItems = JSON.parse(localStorage.get('workings'));

    setCount(countValue);
    setWorkings(workingItems);
  }, [])

  if (!state) {
    return null;
  }

  return (
    <section>
      name: {state.name}<br />
      age: {state.age}<br />
      count: {count}<br />
      working list: {workings.join(',')}
    </section>
  );
};
```

```tsx
// good
interface UserSectionProps {
  name: string;
  age: number;
  count: number;
  workings: string[];
}

export const UserSection: FC<UserSectionProps> = ({
  name,
  age,
  count,
  workings
}) => {
  return (
    <section>
      name: {name}<br />
      age: {age}<br />
      count: {count}<br />
      working list: {workings.join(',')}
    </section>
  );
};
```

### 복잡한 데이터 조작 로직을 가지지 않는다.

표현 컴포넌트 내부에서는 데이터를 가공하거나 사용 데이터의 유효성을 판단하는 등의 로직이 포함되어선 안됩니다.

표현 컴포넌트는 이미 자신이 필요한 만큼 가공된 데이터만을 받아서 사용해야 합니다.

```tsx
// wrong
interface UserListProps {
  // 서버에서 준 데이터가 UI 표현에 적합하지 않다면 이렇게 곧바로 사용하지 않습니다.
  items?: UserDetailEntity[] | null;
  bankDic: Record<string, string>;
}

export const UserList: FC<UserListProps> = ({ items, bankDic }) => {
  if (!items) {
    return null;
  }
  return (
    items
      .filter(item => item.age > 19)
      .map(item => {
        const props = {
          id: item.id,
          name: item?.firstName + ' ' + item.lastName,
          birthDay: toBirthDay(item.birthDate),
          location: getLocationFrom(item.livingLocation, item?.cityName),
          picture: getPictureFrom(item?.profile?.img?.src),
        };

        return props;
      })
      .map((acc, item) => {
        const bankName = acc[item.account] ? findBankName(acc[item.account]) : null;

        return {
          ...item,
          bankName,
        };
      }, bankDic)
      .map(props => (
        <UserDetailItem key={props.id} {...props} />
      ))
  );
};
```

```tsx
// good
interface UserListProps {
  // 그저 주어진대로 사용될 만큼 충분히 가공된 표현 모델만을 사용합니다.
  items: UserDetailModel[];
}

export const UserList: FC<UserListProps> = ({ items }) => {
  return (
    items
      .map(props => (
        <UserDetailItem key={props.id} {...props} />
      ))
  );
};
```

즉 서버에서 내려준 데이터에 구애 받지 않도록, 자신만의 표현 모델을 가지어 사용합니다.

> Q: 그럼 저 복잡했던 데이터 로직들은 어디로 가나요? 🤔
>
> A: 그것들은 모두 store 내 effect 나 action 에서 처리하게 됩니다!

### 사용되는 hooks 나 utility 는 오직 visual logic 만으로 구성 되어야 한다.

유틸리티는 어디서든 간편히 쓸 수 있는 작은 로직이 담긴 함수나 클래스 입니다.

다만 이들을 사용시엔 반드시 순수 함수여야 합니다.

또 한 그 목적은 visual logic 만을 담당 하는 것이어야 합니다.

```tsx
// wrong
interface ChatItemModel {
  id: number;
  chat: string;
  cash: number;
}

interface ChatListProps {
  items: ChatItemModel[];
}

const ChatList: FC<ChatListProps> = ({ items }) => {
  return (
    items.map(item => (
      <ChatItem
        key={item.id}
        // 아래 유틸리티는 단순 순수함수처럼 보이지만...
        right={checkIsMe(item)}
      >
        {item.chat}
        <hr />
        {/* 이 사용법은 훌륭합니다! 👍 */}
        {numberFormat(item.cash)}
      </ChatItem>
    ))
  );
};

// ./util.ts
// 순수함수가 아니라 외부 데이터(쿠키)를 가져와 비교하는 것입니다.
// 이건 순수 함수라 볼 수 없습니다.
function checkIsMe(item: ChatItemModel) {
  const id = getMyIdFromCookie(item);
  return item.id === id;
}

// 단순히 가져온 값을 바꿔 줍니다.
// 외부의 영향을 받지 않으며 그 스스로 격리(isolation)되어 있습니다.
// 즉 이것은 순수 함수 입니다.
function numberFormat(value: number) {
  return value.toLocaleString();
}
```

```tsx
// good
interface ChatItemModel {
  id: number;
  chat: string;
  cash: number;
  // 채팅 내용이 '나'인지 여부를 확인할 수 있는 필드가 추가 되었습니다.
  isMe: boolean;
}

interface ChatListProps {
  items: ChatItemModel[];
}

const ChatList: FC<ChatListProps> = ({ items }) => {
  return (
    items.map(item => (
      <ChatItem
        key={item.id}
        // isMe 라는 별도의 필드를 추가하고 이 값을 그대로 표현에 반영합니다.
        right={item.isMe}
      >
        {item.chat}
        <hr />
        {numberFormat(item.cash)}
      </ChatItem>
    ))
  );
};

// 이하 생략
```

> Q: 그럼 저 isMe 라는 것은 어디서 줘야 할까요? 🤔
>
> A: 그것들은 모두 store 내 effect 나 action 에서 처리하게 됩니다!

## 규칙의 의의

표현 컴포넌트의 코드양을 최소한으로 가져갈 수 있습니다.

복잡한 데이터 로직을 제거하고 오로지 자신만의 표현 모델을 기반으로 렌더링 되기에 그에 부합되는 props 만 잘 넘겨주면 어디서든 재사용 가능합니다.

단순하기에 디버깅이 쉽고 외부 환경에 영향 받지 않으므로 side effect 를 최소화 할 수 있습니다.

또 한 visual test 가 가능하기에 TDD 나 BDD, CDD 로의 개발 운영에 용이해 집니다.