# Naming Conventions: Method

메서드는 일반적으로 특정 클래스나 객체, 컴포넌트의 하위 기능(Function) 입니다.

본 문서는 이러한 메서드 작성 시 빈번히 발생되는 명칭에 대한 가이드 입니다.

## 이벤트 처리기 (Event Handler)

이벤트는 발생 시 그 것을 처리할 별도의 함수가 필요합니다.

이 때 그 함수명은 반드시 접두어로 `handle` 을 붙입니다.

```tsx
// 함수형 컴포넌트 예시

interface Props {
  onClick: () => void;
  onChange: (value: string) => void;
}

const SampleInput: FC<Props> = ({ onClick, onChange }) => {
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.value);
  };

  return (
    <input
      onChange={handleChange}
      {/* 외부 props 이벤트 처리기를 곧바로 쓸 땐 이렇게 바로 삽입 해 줍니다. */}
      onClick={onClick}
    />
  );
}
```

```tsx
// 클래스형 컴포넌트 예시

class SampleInput: Component<Props> {
  construct(props) {
    super(props);
  }
  handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    this.props.onChange(e.target.value);
  }
  render() {
    return (
      <input
        onChange={this.handleChange}
        {/* 외부 props 이벤트 처리기를 곧바로 쓸 땐 이렇게 바로 삽입 해 줍니다. */}
        onClick={this.props.onClick}
      />
    );
  }
}
```

한편, 특별한 사유가 있지 않다면 아래와 같이 곧바로 이벤트에 lambda 식으로 적용시키지 않습니다.

```tsx
// wrong

const SampleInput: FC<Props> = ({ onChange }) => {
  return (
    <input
      {/* 이런 방법은 가급적 지양 합시다. */}
      onChange={(e: ChangeEvent<HTMLInputElement>) => {
        onChange(e.target.value);
      }}
    />
  );
};
```

만약 `items` 같은 배열형 자료로 순회 할 경우엔 아래와 같이 `currying` 을 사용 합니다.

이 때 커링 사용 시 아래와 같이 접미사로 `Curried`를 붙여줍니다.

```tsx
interface ProdItem {
  id: number;
  name: string;
  price: number;
}

interface Props {
  items: ProdItem[];
  onBuying: (item: ProdItem) => void;
}

const SampleList: FC<Props> = ({ items, onBuying }) => {
  // 커링을 이용하는 핸들러이기 때문에 'Curried'를 붙여 주었습니다.
  const handleBuyingCurried = (item: ProdItem) => () => {
    onBuying(item);
  };

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          상품명: {item.name}<br/>
          가격: {item.price}원
          {/* 적용 시 해당 Loop 의 item 을 한번 넣어줍니다. */}
          <button onClick={handleBuyingCurried(item)}>
        </li>
      ))}
    </ul>
  );
};
```

### 규칙의 의의

이벤트를 발생시키는 쪽과 그 것을 처리하는 것의 명칭을 명확히 하여 코드의 가독성을 높이는 것이 주요 목적 입니다.

또 한 곧바로 lambda 사용을 지양하는 이유는 내부 마크업과 이벤트 처리부를 따로 두어 `관심사 분리를 유도` 하기 위함 입니다.

이렇게 분리 할 경우 이벤트 처리기 수정 시 핸들러만 보면 되며, 반면 마크업 변경은 마크업 쪽만 건드리면 됩니다.
