# Coding Conventions: Component

본 문서는 React 를 이용한 UI 컴포넌트 코딩 시 지켜야 할 규칙에 대하여 설명합니다.

## render 함수 금지

React 로 컴포넌트 작성 시 내용이 비대해져서 불가피하게 아래와 같이 `render` 함수를 내부적으로 선언하여 사용하는 경우가 있습니다.

```tsx
// Wrapper.tsx
interface Props {
  title: string;
  desc: string;
  imgSrc: string;
  onClick: () => void;
}

const Wrapper: FC<Props> = ({ onClick, title, desc, imgSrc }) => {
  const renderDesc = () => {
    return (
      <>
        <h3>{title}</h3>
        <p>{desc}</p>
      </>
    );
  };

  return (
    <section>
      <img src={imgSrc} alt="고래가 춤을 춥니다" />
      <div>
        {renderDesc()}
      </div>
      <button type="button" onClick={onClick}>팔로우</button>
    </section>
  );
};
```

이럴 땐 별도의 컴포넌트로 분리하여 사용합니다.

```tsx
// 파일을 분리 합니다.
// SubDesc.tsx
interface Props {
  title: string;
  desc: string;
}
// 기존 클로저로 받던 속성값을 자체 props 로 선언하여 받게 하였습니다.
export const SubDesc: FC<Props> = ({ title, desc }) => {
  return (
    <>
      <h3>{title}</h3>
      <p>{desc}</p>
    </>
  );
};

// Wrapper.tsx
interface Props {
  title: string;
  desc: string;
  imgSrc: string;
  onClick: () => void;
}

const Wrapper: FC<Props> = ({ onClick, title, desc, imgSrc }) => {
  return (
    <section>
      <img src={imgSrc} alt="고래가 춤을 춥니다" />
      <div>
        {/* 이젠 자신이 가진 속성값을 넘기기만 하면 됩니다. */}
        <SubDesc title={title} desc={desc} />
      </div>
      <button type="button" onClick={onClick}>팔로우</button>
    </section>
  );
};
```

### 조금 더 개량 해 보기

이렇게 컴포넌트를 쪼개다보면, 하위 컴포넌트는 상위 컴포넌트가 가진 속성(props)의 일부(subset)를 자연스레 가지게 됩니다.

그럼 상위 컴포넌트의 속성은 하위 컴포넌트가 가진 속성들을 조합 한 결과물이 될 것입니다.

즉, 아래와 같이 interface 상속을 활용 하면 좋습니다.

```ts
// SubDesc.tsx
export interface SubDescProps {
  title: string;
  desc: string;
}

// Wrapper.tsx
interface Props extends SubDescProps {
  imgSrc: string;
  onClick: () => void;
}
```

이들 Props 를 활용한 컴포넌트는 다음과 같습니다.

```tsx
// SubDesc.tsx
// 선언된 props 를 외부에 공유 시키기 위해 이름이 변경 되었습니다.
export const SubDesc: FC<SubDescProps> = ({ title, desc }) => {
  return (
    <>
      <h3>{title}</h3>
      <p>{desc}</p>
    </>
  );
};

// Wrapper.tsx
const Wrapper: FC<Props> = ({ onClick, imgSrc, ...props }) => {
  return (
    <section>
      <img src={imgSrc} alt="고래가 춤을 춥니다" />
      <div>
        {/*
          하위 컴포넌트의 props 는 상위 컴포넌트
          props 의 subset 이므로 따로 묶어서 spread 로 넘겨주면
          보다 코딩이 깔끔해 집니다!
        */}
        <SubDesc {...props} />
      </div>
      <button type="button" onClick={onClick}>팔로우</button>
    </section>
  );
};
```

### render 함수를 왜 따로 만들게 되었을까요?

과거에 class 컴포넌트로 작성했던 시기, `render` 함수의 내용이 많아져서 별도의 렌더링 함수를 만들어 사용 하던 것이 시초 입니다.

```tsx
// 과거의 클래스형 컴포넌트 예시
class PastSection extends React.Component<Props> {
  constructor(props) {
    super(props);
  }

  renderDesc() {
    const {
      title,
      desc,
    } = this.props;

    return (
      <>
        <h3>{title}</h3>
        <p>{desc}</p>
      </>
    );
  }

  render() {
    const {
      imgSrc,
      onClick,
    } = this.props;

    return (
      <section>
        <img src={imgSrc} alt="고래가 춤을 춥니다" />
        <div>
          {this.renderDesc()}
        </div>
        <button type="button" onClick={onClick}>팔로우</button>
      </section>
    );
  }

}
```

현재 함수 컴포넌트를 만들때도, 옛날 클래스 컴포넌트를 만들때도,

중복되는 렌더링이나 너무 많아진 마크업을 한 곳에 다 몰아넣다보니 가독성이 떨어져서 이러한 업무 패턴이 생기게 되었습니다.

그래서 별도의 서브 렌더링 메서드를 만들어 사용 했습니다.

그것이 이 곳에서 언급하는 `render` 함수의 시초격이라 볼 수 있습니다.

### 규칙의 의의

컴포넌트 외부에 렌더링 함수를 두어 매번 렌더링 될 때 마다 함수가 재생성 되는 것을 막을 수 있습니다.

그리고 그 렌더링 함수를 컴포넌트화 함으로써 필요 시 재사용이 가능해집니다.

또 한 상위/하위 컴포넌트가 분리되어 각자 서로의 기능에 대한 책임만 지면 되므로 자연스레 관심사 분리를 유도할 수 있습니다.

덤으로 코드 가독성도 좋아집니다.