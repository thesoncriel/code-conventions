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

