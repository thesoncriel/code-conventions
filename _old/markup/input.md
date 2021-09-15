# Code Conventions: Markup - Input

웹개발 시 사용되는 마크업 언어인 HTML 작성 시 웹표준을 준수하며 작업하는 방법을 설명 합니다.

## Checkbox

체크박스는 일반적으로 `<input type="checkbox">` 요소를 사용합니다.

이 때 `<label>` 요소를 함께 사용하면 마우스 클릭 시 체크박스 UI와 근처 텍스트에 대하여 반응이 됩니다.

```html
<label>
  <input type="checkbox" />
  체킷체킷!
</label>
```

만약 불가피하게 레이블이 떨어진 경우엔 `id` 를 두고 연관된 레이블에 `for` 값을 채워 보완할 수 있습니다.

```html
<table>
  <tbody>
    <tr>
      <th>
        <label for="chk">놓치지 않을거에요!</label>
      </th>
      <td>
        <input type="checkbox" id="chk" />
      </td>
    </tr>
  </tbody>
</table>
```

### 응용

최신 웹브라우저 기준, checkbox 에 대해선 `appearance` 속성으로 기본 표현 방법을 풀어버리고 custom style 이 적용 가능 합니다.

```scss
.restyled {
  -webkit-appearance: none;
  display: inline-block;
  vertical-align: middle;
  background: #ddd;
  border-radius: 14px;
  border: none;
  width: 20px;
  height: 20px;
  &:focus {
    outline: none;
  }

  &:checked::after {
    content: '\2713';
    font-size: 1.5em;
  }
}
```

```html
<label>
  <input type="checkbox" class="restyled" />
  스타일이 바뀐 체크박스
</label>
```

그러나 일반적으로 체크박스를 표현할 때 별도의 이미지 아이콘을 많이 사용하는 까닭에 이 방법엔 한계가 있습니다.

더군다나 기기마다 표현 방법이 다를 수 있습니다. 😱

때문에 아래와 같이 input 요소는 숨겨서 표현 합니다.

```scss
.redesigned {
  & > input {
    display: block;
    width: 1px;
    height: 1px;
    // 투명도를 0으로 하지 않는 까닭:
    // 기기마다 렌더링 방식이 달라서 display: none 하거나 완전 투명하게 하면
    // Hit-box 가 사라지는 경우가 종종 있기 때문에
    // 거의 안보일 정도의 반투명으로 둡니다.
    opacity: 0.01;
    position: absolute;

    &:focus {
      outline: none;
    }

    & ~ .checked {
      display: none;
    }
    // 2개의 아이콘을 두어 체크된 상태를 조건으로 보임/숨김 처리를 합니다.
    &:checked {
      & ~ .unchecked {
        display: none;
      }
      & ~ .checked {
        display: inline-block;
      }
    }
}
```

```html
<label class="redesigned">
  <!-- 입력 요소는 숨겨지며 오직 핸들러와 상태를 가지는 역할만 맡습니다. -->
  <input type="checkbox" />
  <!-- 실제 사용되는 체크버튼 아이콘. css 에 의해 상태 핸들링 됩니다. -->
  <i class="icon unchecked"></i>
  <i class="icon checked"></i>
  <span>이미지 적용 체크박스</span>
</label>
```

아이콘의 css 클래스 상태에 따라 내용이 변경 가능하다면, 다음과 같이 스크립트 핸들링으로도 가능 합니다.

```scss
.redesigned {
  & > input {
    display: block;
    width: 1px;
    height: 1px;
    opacity: 0.01;
    position: absolute;

    &:focus {
      outline: none;
    }
}
```

```html
<label class="redesigned">
  <!-- 입력 요소는 숨겨지며 오직 핸들러와 상태를 가지는 역할만 맡습니다. -->
  <input type="checkbox" onchange="handleChange(event)" />
  <!-- 실제 사용되는 체크버튼 아이콘. 하나만 사용하며 js를 통해 핸들링 합니다. -->
  <i class="icon unchecked"></i>
  <span>이미지 적용 체크박스</span>
</label>
```

```js
function handleChange(event) {
  const icon = document.querySelector('.icon');

  icon.classList.remove('unchecked');
  icon.classList.remove('checked');
  icon.classList.add(event.target.checked ? 'checked' : 'unchecked');
}
```

리액트일 경우엔 다음과 같습니다.

```tsx
interface Props {
  checked?: boolean;
  onChange?: (checked: boolean) => void;
}
const Checkbox = styled.input`
  display: block;
  width: 1px;
  height: 1px;
  opacity: 0.01;
  position: absolute;

  &:focus {
    outline: none;
  }
`;

// 체크된 상태를 외부에서 주는 props 에 의존할 때의 예시 입니다.
const SampleCheckbox: FC<Props> = ({ checked = false, onChange }) => {
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    onChange && onChange(e.target.checked);
  };

  return (
    <label>
      <Checkbox checked={checked} onChange={handleChange} />
      {/* 아이콘에 checked 와 unchecked 2 종류가 있다 가정합니다. */}
      <i className={classnames('icon', { checked, unchecked: !checked })}>
    </label>
  );
};
```

### 응용: Radio Button

라디오 버튼은 `<input type="radio>` 를 이용하되 기본적인 사용법은 checkbox 과 거의 같습니다.

다른점은 아래와 같이 2가지 입니다.

1. 반드시 `<form>` 요소로 감싼 곳에서 사용
2. 각 `<input>` 요소에 name 과 value 를 미리 지정 할 것.

#### form 에 감싸고 name 과 value 를 설정해야 하는 이유

다른 입력 요소 (input element)들도 마찬가지지만, 라디오 버튼은 여러개 중 단 1개를 선택하는 UI 입니다.

이에 대해 웹표준 방식으로 제어하기 위해선 form 안에 input을 넣고 동작 영역(scope)을 한정 지어야 합니다.

그리고 서로 연관된 라디오 버튼들은 `name` 속성에 같은 값으로 지정해줘야 합니다.

이 상태에서 라디오 버튼 클릭 시, 같은 이름의 라디오 버튼일 경우 하나만 선택되는 동작이 자연스레 이뤄집니다.

마지막으로 `value`는 선택했을 때 전달될 값 입니다.

상기 내용을 바탕으로 작성된 일반적인 마크업으론 아래와 같습니다.

```html
<form>
  <label> <input type="radio" name="fruit" value="apple" /> 사과</label>
  <label>
    <input type="radio" name="fruit" value="pineapple" /> 파인애플
  </label>
  <label> <input type="radio" name="fruit" value="tomato" /> 토마토</label>
</form>
```

리액트로 변화된 값을 받아오는 예시는 다음과 같습니다. (스타일은 생략)

```tsx
interface Props {
  value?: string;
  onChange?: (value: string) => void;
}

const RadioList: FC<Props> = ({ value, onChange }) => {
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    onChange && onChange(e.target.value);
  };

  return (
    <form>
      <label>
        <input
          type="radio"
          name="fruit"
          value="apple"
          checked={value === 'apple'}
        />{' '}
        사과
      </label>
      <label>
        <input
          type="radio"
          name="fruit"
          value="pineapple"
          checked={value === 'pineapple'}
        />{' '}
        파인애플
      </label>
      <label>
        <input
          type="radio"
          name="fruit"
          value="tomato"
          checked={value === 'tomato'}
        />{' '}
        토마토
      </label>
    </form>
  );
};
```

### 규칙의 의의

체크 박스 계통의 UI 구성 시 div 나 button 으로만 구성하는 경우가 많습니다.

이러한 방법을 지양하고 웹표준 방법을 통해 올바르게 구성하는 방법을 작업에 적용하는 것이 주된 목적 입니다.

또 한 웹표준 방법을 사용하면, DOM 의 checked 상태를 직접적으로 알 수 있기 때문에 자동 테스트를 구성할 때도 용이합니다.

### jsfiddle 샘플 예제

- [checkbox](https://jsfiddle.net/u3n926mp/6/)
- [radiobutton](https://jsfiddle.net/yhtmzeus/)
