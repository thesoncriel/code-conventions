# Component: Event Props

## 이벤트 Props

UI 컴포넌트는 다양한 기능을 가지고 있고, 그 중 하나가 바로 `이벤트` 입니다.

리액트는 이벤트를 Function Type 의 props 로 받아들여 처리합니다.

이 때 그 이벤트 props 는 반드시 접두어로 `on` 을 붙여줍니다.

```tsx
// props 타입 선언
interface Props {
  name: string;
  onClick: () => void;
  onChange: (value: string) => void;
}

// 적용 예시
const SampleInput: FC<Props> = ({ name, onClick, onChange }) => {
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.value);
  };

  return (
    <>
      명칭: {name}
      <br />
      <input onChange={handleChange} onClick={onClick} />
    </>
  );
};
```

## DOM Event Object 직접 전달 금지

이벤트 props 는 다음과 같이 특수한 상황이 아니라면, 본래 DOM 이 가지던 Event Object 를 그대로 반환하지 않습니다.

```tsx
// wrong
interface Props {
  // 과연 외부에선 MouseEvent 라는 복잡한 객체를 필요로 할까요?
  onClick: (event: MouseEvent<HTMLElement>) => void;
  onChange: (event: ChangeEvent<HTMLInputElement>) => void;
}

export const SampleInput: FC<Props> = ({ onClick, onChange }) => {
  return (
    <>
      <input onChange={onChange} />
      <button type="button" onClick={onClick}>
        적용
      </button>
    </>
  );
};
```

상기 예시 처럼 작성할 시 `<SampleInput>` 을 사용하는 곳에서는 이벤트 인자(Event Arguments)에 대해서 너무 많은 것을 알아야 합니다.

```tsx
import { SampleInput } from './SampleInput';

const Client: FC = () => {
  const [value, setValue] = useState('');

  // 이벤트 객체를 전달하지만, 전혀 사용하지 않습니다.
  const handleClick = (event: MouseEvent<HTMLElement) => {
    alert('클릭!');
  };

  // 이벤트 객체를 통째로 주고 있지만
  // 이 곳에서 쓰고자 하는건, 사용자가 변경한 값 뿐입니다!
  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {
    setValue(event.target.value);
  };

  return (
    <>
      입력값: {value}<br />
      <SampleInput onClick={handleClick} onChange={handleChange} />
    </>
  );
};
```

이러한 패턴은 관심사 분리 측면에서 바람직하지 않습니다.

상기와 같은 코드는 다음과 같이 변경합니다.

```tsx
// good
interface Props {
  // 단순히 클릭 여부만 전달하기에 event argument 는 생략합니다.
  onClick: () => void;
  // 사용자의 변경 자료만 전달합니다.
  onChange: (value: string) => void;
}

const SampleInput: FC<Props> = ({ onClick, onChange }) => {
  // 이벤트 핸들러는 외부에서 관심가질만한 데이터만 전달 합니다.
  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {
    onChange(event.target.value);
  };

  return (
    <>
      <input onChange={handleChange} />
      <button type="button" onClick={onClick}>
        적용
      </button>
    </>
  );
};

// 사용 예시
import { SampleInput } from './SampleInput';

const Client: FC = () => {
  const [value, setValue] = useState('');

  const handleClick = () => {
    alert('클릭!');
  };

  // 전달 받은 값을 그대로 사용합니다.
  const handleChange = (argValue: string) => {
    setValue(argValue);
  };

  return (
    <>
      입력값: {value}
      <br />
      <SampleInput onClick={handleClick} onChange={handleChange} />
    </>
  );
};
```

### Custom Event Arguments

아래는 유사한 사례로, 마우스 위치 추적(Position Tracking) 예시 입니다.

외부에서는 Mouse Event Object 를 원할 것 같지만, 실제론 좌표만 알아도 충분합니다.

```tsx
// good
// 참고: 이벤트 전달인자는 가급적 별도 파일에 선언하여 사용합니다.
export interface CursorPositionArgs {
  posX: number;
  posY: number;
}

interface Props {
  onCursorPositionChange: (args: CursorPositionArgs) => void;
}

export const MouseCursorTracker: FC<Props> = ({ onCursorPositionChange }) => {
  const handleMouseMove = (event: MouseEvent<HTMLElement>) => {
    onCursorPositionChange({
      posX: event.movementX,
      poxY: event.movementY,
    });
  };

  return <StyledSection onMouseMove={handleMouseMove} />;
};

// 사용 예시
import { MouseCursorTracker, CursorPositionArgs } from './MouseCursorTracker';

const Client: FC = () => {
  const [pos, setPos] = useState<CursorPositionArgs>({ posX: 0, posY: 0 });

  const handleCursorPositionChange = (args: CursorPositionArgs) => {
    setPos(args);
  };

  return (
    <div>
      마우스 위치: {pos.posX}, {pos.posY}
      <StyledSection onCursorPositionChange={handleCursorPositionChange} />
    </div>
  );
};
```

이렇게 코딩하면 사용처에서는 그들이 필요한(혹은 관심 있어하는) 이벤트 인자만 받아들이므로 사용도 간단하고 코드양도 줄어듭니다.

## Event Handler 에 Setter 와 Dispatcher 전달 금지

이벤트 props 는 useState 의 setter 나 redux 같은 상태 관리자의 dispatcher 를 직접적으로 전달하지 않습니다.

아래는 잘못된 예시 입니다.

```tsx
// wrong
interface Props {
  value: string;
  // oops!!
  onChange: React.Dispatch<React.SetStateAction<string>>;
}
export const MyComponent: FC<Props> = (props) => {
  // 생략..
};

// 사용처 예시
const Client: FC = () => {
  const [value, setValue] = useState('');

  return <MyComponent value={value} onChange={setValue} />;
};
```

MyComponent 는 선언된 `onChange` 타입 때문에 외부 환경에 관심이 많을 수 밖에 없습니다. 따라서 `onChange` 이벤트는 반드시 `useState` 에서 비롯된 settter 를 바라보게 되는 문제가 생깁니다.

이런 상황은 아래와 같이 변경시켜 줍니다.

```tsx
// good
interface Props {
  value: string;
  // 이벤트 타입을 직접 명시해줍니다.
  onChange: (value: string) => void;
}
export const MyComponent: FC<Props> = (props) => {
  // 생략..
};

// 사용처 예시
const Client: FC = () => {
  const [value, setValue] = useState('');

  // 이벤트는 반드시 핸들러를 통하여 수행합니다.
  const handleChange = (args: string) => {
    setValue(args);
  };

  return <MyComponent value={value} onChange={handleChange} />;
};
```

## 규칙의 의의

그 어떤 코드에서도 `on` 접두어가 보인다면, 이를 통해 해당 props 가 이벤트임을 간접적으로 알게 하는 단서가 됩니다.

이벤트 인자(Event Argument)는 외부에 공개하고 싶은 만큼 충분히 걸러진 자료로 전달합니다.

이를 통하여 사용처(client)에서는 원하는 것만 받는 이점이 생기고 UI 컴포넌트 당사자는 자신의 모든 것을 알리지 않아도 됩니다.

즉, 이벤트 영역에서 캡슐화(encapsulation)와 `관심사 분리`가 이뤄질 수 있습니다.

setter 나 dispatcher 를 event handler 에서 배제함으로써 컴포넌트가 사용될 외부 환경에 보다 독립적으로 운용될 수 있고 재사용성이 증가됩니다.
