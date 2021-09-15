# Code Conventions: Markup - Button and Anchor

웹개발 시 사용되는 마크업 언어인 HTML 작성 시 웹표준을 준수하며 작업하는 방법을 설명 합니다.

## Button

버튼은 사용자와의 상호작용(Interaction)을 위한 입력 요소중 하나 입니다.

이 중 Click 과 같이 매우 기본적인 상호작용을 맡고 있습니다.

버튼은 일반적으로 아래와 같이 `<button type="button">` 요소를 사용합니다.

```html
<button type="button">클릭 해 보아요!</button>
```

클릭에 대한 상호작용에 button 대신 `<input type="button" value="레이블">` 요소를 사용할 수 있으나 이 요소는 자식요소(children)를 허용하지 않기 때문에 스타일링이 제한적입니다.

따라서 특별한 사유가 있지 않는 한 오로지 버튼은 `<button>` 요소로만 사용합니다.

### 클릭이 필요하다면 버튼으로 쓰세요

특별한 사유가 없다면, 사용자의 터치나 클릭에 반응하는 요소는 `button` 으로만 사용합니다.

div 나 a, span 등 목적이 다르거나 범용적인 레이아웃 목적의 요소는 본래 그 역할에 충실하도록 마크업 합니다.

```html
<!-- wrong -->
<div onclick="handleClick()">클릭 합시다!</div>
<!-- wrong -->
<a onclick="handleClick()">클릭 합시다!</a>

<!-- good -->
<button type="button" onclick="handleClick()">클릭 합시다!</button>
```

### type=button 의 의미

기본적으로 button 요소의 type 은 `submit` 입니다.

그래서 단순 버튼으로 쓰려면 `type="button"` 으로 변경 해 주어야 합니다.

이렇게 된 까닭에는, 웹표준 마크업에서 기본적으로 쓰이는 `form` 요소와 함께 사용해야 하기 때문입니다.

만약 `type="button"` 으로 바꿔주지 않을 경우, 클릭 시 가장 가까운 `form` 요소를 찾아 그 요소의 `action` 에 지정된 경로를 수행하려 합니다.

만약 비동기 양식이라 action 이 지정되어 있지 않다면, 현재 페이지 경로를 기준으로 보내기(submit)를 합니다.

그래서 아래와 같은 방식으로 코딩을 해 본 경험이 있을 겁니다.

```html
<form>
  <!-- 기본적으로 type=submit 입니다 -->
  <button id="btn">클릭! 클릭!</button>
</form>
```

```js
const elem = document.getElementById('btn');

elem.addEventListener('click', (e) => {
  // 해당 요소의 기본 기능을 매번 막아 주어야 합니다!
  e.preventDefault();
  // 슬프게도, 버튼의 기본 기능을 막지 않으면 이 기능이 수행되지 않습니다.
  anyProcessing();
});
```

이 것을 사용될 버튼의 type을 `button` 으로 변경 시 불필요한 예외 코드가 줄어듭니다.

```html
<form>
  <button id="btn" type="button">클릭! 클릭!</button>
</form>
```

```js
const elem = document.getElementById('btn');

elem.addEventListener('click', (e) => {
  // 이젠 그냥 수행해도 됩니다.
  anyProcessing();
});
```

### form 요소와의 관계

button 이나 input, textarea 와 같이 사용자와의 상호작용을 위한 요소는 `form` 요소 내에 작성하여 사용하는 것이 일반적입니다.

필요에 따라 form 요소 바깥에 두고 사용 할 수 있습니다만, 권장되는 사항은 아닙니다. 🙂

## Anchor

한편, 하이퍼링크(hyper-link) 기능을 맡는 `<a>` 요소는 `href` 속성을 통해 클릭 시 다른 페이지, 혹은 다른 웹사이트로 이동시켜줄 수 있습니다.

```html
<a href="other-page.html"> 다른 페이지로 가즈아~! </a>
```

리액트로 SPA 구성 시 `react-router` 에서 제공 해 주는 `<Link>` 컴포넌트를 적극 활용합니다.

```tsx
import { Link } from 'react-router-dom';

const SampleSection: FC = () => {
  return (
    <div>
      회원가입 페이지로 가즈아!
      <br />
      <Link to="/signup">회원가입 바로가기</Link>
      <a href="https://www.gowid.com" target="_blank">
        고위드 홈페이지로 가기
      </a>
    </div>
  );
};
```

### 규칙의 의의

클릭 가능한 요소를 `<a>` 나 `<span>`, `<div>` 등으로 일관성 없이 사용하는 것을 지양합니다.

그리고 웹표준에 맞추어 이들이 제공하는 기본 기능을 제대로 사용하기 위함 입니다.

### 참고

> The elements used to create controls generally appear inside a FORM element, but may also appear outside of a FORM element declaration when they are used to build user interfaces. This is discussed in the section on intrinsic events. Note that controls outside a form cannot be successful controls.
>
> - [W3 HTML form controls](http://www.w3.org/TR/html401/interact/forms.html#form-controls)

> 해석: 이런 요소들은 일반적으로 `form` 요소내에서 만들어 사용합니다, 다만 UI 작성 시 종종 `form` 요소 바깥에 선언하여 사용 합니다. 이 것은 본질적인 이벤트 섹션에서 논의 되었습니다. `form` 요소 바깥의 입력 요소는 성공적인 컨트롤이 될 수 없습니다.
