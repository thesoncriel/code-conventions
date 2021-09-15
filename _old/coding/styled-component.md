# Coding Conventions: Styled Component

본 문서는 React 와 `styled-component(이하 SC)` 이용 시 알아두어야 할 규칙에 대하여 설명 합니다.

## 단순 nesting 금지

SC는 자체적으로 className 을 가지므로 아래와 같이 SCSS 처럼 네스팅을 활용하는 것은 지양합니다.

```tsx
// before
const Client: FC = () => {
  return (
    <Wrap>
      <div>
        <input className="input-text" />
        <span>입력하세요!</span>
      </div>
    </Wrap>
  );
};

const Wrap = styled.section`
  margin: 0 auto;

  & > div {
    display: inline-block;
    border: 1px solid #888;

    span {
      display: block;
      text-decoration: underline;
      color: #800;
    }
  }
  .input-text {
    background: #fff;
    border: 1px solid #ddd;
    border-radius: 5px;
  }
`;
```

이렇게 된 경우 다음과 같이 작성 합니다.

```tsx
// after

const Client: FC = () => {
  return (
    <Wrap>
      <InnerWrap>
        <InputText/>
        <GuidanceMessage>입력하세요!</GuidanceMessage>
      </InnerWrap>
    </Wrap>
  );
};

const Wrap = styled.section`
  margin: 0 auto;
`;
const InnerWrap = styled.div`
  display: inline-block;
  border: 1px solid #888;
`;
const InputText = styled.input`
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 5px;
`;
const GuidanceMessage = styled.span`
  display: block;
  text-decoration: underline;
  color: #800;
`;
```

### 규칙의 의의

스타일 코드가 네스팅에 독립적이면 각 컴포넌트별 스타일 코드의 길이가 짧아집니다.

그리고 특정 스타일은 그 컴포넌트만이 책임지기 때문에 필요하다면 다른 곳에서도 쓸 수 있는 재활용성이 증가 됩니다.

또 한 각 컴포넌트는 단순 마크업 명칭(div, span 등)이 아닌 각자 기능에 따라 의미부여가 되므로 컴포넌트의 역할과 코드 가독성도 증가 됩니다.

## 스타일 코드 위치

SC 사용 시 스타일 코드를 둘 때는 기본적으로 `사용되는 컴포넌트 내부의 아랫쪽`에 둡니다.

```tsx
export const ClientSection: FC = () => {
  return (
    <StyledWrap>
      <Label htmlFor="sel_client">상품 선택하기</Label>
      <Selectbox id="sel_client">
        <option value="">없음</option>
        <option value="money">떡값</option>
        <option value="gift_tuna">참치 선물세트</option>
        <option value="gift_spam">스팸 선물세트</option>
      </Selectbox>
    </StyledWrap>
  );
};

const StyledWrap = styled.section`
  margin: 0 auto;
  font-size: 12px;
`;

const Label = styled.label`
  display: block;
`;

const Selectbox = styled.select`
  display: block;
  width: 100%;
  height: 20px;
`;
```

### 스타일 컴포넌트를 외부에 두기

만약 스타일 코드가 많아 부득이하게 외부에 두고 싶다면, 아래와 같이 현재 컴포넌트 명칭 뒤에 `.style` 접미어를 붙인 파일에 선언하여 사용합니다.

그리고 확장자도 **.tsx** 가 아닌 `.ts` 만 사용합니다.

| 컴포넌트 파일명 | 스타일 파일명 |
| :---------- | :--------- |
| ClientSection.tsx      | SlientSection.style.ts      |
| UserInfoModifyForm.tsx | UserInfoModifyForm.style.ts |

참고로 이러한 파일을 선언 했을 때 내부의 코드는 오로지 `스타일 코드`만 존재 해야 합니다.

```ts
// ArticleSection.style.ts
// 허용되는 코드
const cssFlexBlock = css`
  display: flex;
  width: 100%;
`;

export const ArticleWrap = styled.article`
  ${cssFlexBlock}
  color: #888;
  font-size: 14px;
`;

export const FormGroup = styled.div`
  ${cssFlexBlock}
  border-bottom: 1px dotted #aaa;
`;
```

그래서 내부에 아래와 같이 React 의 특성을 이용한 컴포넌트 로직 코드가 존재해선 안됩니다.

```tsx
// SelectionModal.style.ts
// 이건 허용됩니다.
export const ModalWrap = styled.div`
  position: fixed;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.2);
  transition: background 0.2s;
`;

// but.. 이건 안됩니다.
// 컴포넌트는 eslint 에서 tsx 만 허용토록 거르기 때문에 애초에 불가 할 것입니다.
export const ExternalModal: FC<ExternalProps> = (props) => {
  if (!props.show) {
    return null;
  }
  return (
    <ModalWrap>
      {props.children}
    </ModalWrap>
  );
};
```

또 한 SC의 props 를 이용한 로직이 포함되어서도 안됩니다.

```ts
// UserForms.style.ts
// 이건 허용됩니다.
export const InputNumber = styled.input`
  border: 1px solid #333;
  border-radius: 5px;
  padding: 5px 8px;
  font-size: 14px;
`;

// but.. 이건 안됩니다.
interface AlertTextProps {
  $show?: boolean;
  $color?: 'red' | 'orange' | 'green' | 'black';
}

export const AlertText = styled.div<AlertTextProps>`
  display: ${({ props }) => props.$show ? 'block' : 'none'};
  color: ${({ props }) => props.$color || 'inherit' };
`;
```

이렇게 별도 로직이 들어가거나 props 를 통한 변경이 있는 컴포넌트는 그 기능을 사용하는 컴포넌트 파일 내에 둡니다.

만약 여기저기서 사용되는 것이라면 `feature module` 내에서 공용으로 쓰일 수 있기에 별도로 분리 하여 사용합니다. (예: common 등)

### 외부로 분리된 컴포넌트 사용

2가지 방법이 있습니다.

1. `S` 라는 명칭의 컴포넌트 뭉치를 이용하기
    - 코드가 간결해서 좋습니다.
    - 그러나 XML 의 하위 호환인 JSX 마크업에 점(.)이 들어가는 것에 호불호가 있을 수 있습니다.
    - 사용하지 않는 스타일 컴포넌트가 존재할 시 Tree-Shaking 의 이득을 보지 못합니다.
2. 필요한만큼 import 하여 사용하기
    - 안쓰이는 컴포넌트가 있어도 Tree-Shaking 의 이점을 볼 수 있습니다.
    - 마크업이 깔끔해 집니다.
    - 그러나 import 구문이 길어질 수 있습니다.

```tsx
// 1. `S` 라는 명칭의 컴포넌트 뭉치를 이용하기
import * as S from './ClientComp.style';

export const ClientComp: FC = () => {
  return (
    <S.ClientWrap>
      <S.Heading>
        <S.Logo src="/public/logo.png" />
        제목제목..
      </S.Heading>
      <S.Desc>내용내용</S.Desc>
    </S.ClientWrap>
  );
};
```
```tsx
// 2. 필요한만큼 import 하여 사용하기
import {
  ClientWrap,
  Heading,
  Logo,
} from './ClientComp.style';

export const ClientComp: FC = () => {
  return (
    <ClientWrap>
      <Heading>
        <Logo src="/public/logo.png" />
        제목제목..
      </Heading>
      <Desc>내용내용</Desc>
    </ClientWrap>
  );
};
```

둘 중 상황에 따라 선택하여 사용하면 됩니다. 🙂

### 규칙의 의의

스타일 컴포넌트를 두는 장소에 일관성을 두어 약속된 장소에서 관련 코드를 쉽게 찾고자 함이 주요 목적 입니다.

그리고 연관된 스타일이 쓰이지 않을 경우, 이름과 그에 대한 관리적 측면에서도 용이합니다.

이는 스타일링 내용이 실제 사용되는 컴포넌트 내부에 있을 때, 응집성(cohension)을 높일 수 있기에 설계적 측면에서 바람직한 방법입니다.

> **응집성**
>
> 연관된 것 끼리 뭉쳐있는 특성. 결합성(coupling)과는 반대 개념 입니다.