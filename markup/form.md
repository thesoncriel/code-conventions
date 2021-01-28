# Code Conventions: Markup - Form

웹개발 시 사용되는 마크업 언어인 HTML 작성 시 웹표준을 준수하며 작업하는 방법을 설명 합니다.

## Form

입력 양식은 일반적으로 `<form>` 요소와 함께 쓰이는 것을 권장 합니다.

```html
<!-- 기존에 이랬다면... -->
<div class="form-container">
  <label for="txt_name">이름: </label>
  <input type="text" name="name" id="txt_name" /><br />
  <label for="sel_gender">성별: </label>
  <select name="gender" id="sel_gender">
    <option value="mal">남성</option>
    <option value="femal">여성</option>
  </select>
</div>
```

```html
<!-- 이젠 이렇게..! -->
<form class="form-container">
  <label for="txt_name">이름: </label>
  <input type="text" name="name" id="txt_name" /><br />
  <label for="sel_gender">성별: </label>
  <select name="gender" id="sel_gender">
    <option value="mal">남성</option>
    <option value="femal">여성</option>
  </select>
</form>
```

### 입력 후 엔터

`<input>` 요소를 통하여 입력 후 엔터를 치면 서버에 전달(submit)되길 기대하는 UX가 굉장히 많습니다.

잘 모를 경우 일일이 key-event 에서 엔터값을 잡는 등의 수고를 들이는 경우가 있는데,

이럴 땐 아래와 같이 `<form>` 요소로 감싼 상태에서 `<button type="submit">`을 함께 두면 해결 됩니다.

```html
<form class="form-container" onsubmit="handleSubmit(event)">
  <label for="txt_name">이름: </label>
  <input type="text" name="name" id="txt_name" /><br />
  <label for="num_age">나이: </label>
  <input type="number" name="age" id="num_age" /><br />
  <!--
    서밋 버튼을 클릭한다는 것은 곧 form 의 submit 을 수행한다는 의미이므로
    별도 click 이벤트를 주지 않습니다.
  -->
  <button type="submit">보내기</button>
</form>
```

```js
function handleSubmit(event) {
  // 비동기로 보낼 땐 반드시 기본 기능을 중지 시켜야 페이지 전환을 막을 수 있습니다.
  event.preventDefault();
  doSomeAsyncLogic();
}
```

만약 보내기 버튼이 별도로 존재하지 않고, 그저 엔터 이벤트만 취하고 싶다면 아래와 같이 서밋 버튼에 대한 숨김 스타일을 추가하면 됩니다.

```scss
.form-container {
  & > button[type='submit'] {
    position: absolute;
    left: 0;
    bottom: 0;
    display: block;
    width: 1px;
    height: 1px;
    overflow: hidden;
    opacity: 0.01;
  }
}
```

참고로 이럴 경우, 어차피 안보이기에 button 이 아닌 input[type=submit] 으로 사용해도 됩니다.

- [피들러 예제](https://jsfiddle.net/0L9vro7t/)

### 라디오 버튼 동작

`<input type="radio">`가 `<form>` 요소 내에 있을 경우 동작 방식을 의도대로 제어할 수 있습니다.

자세한건 [markup/input](./input.md) 내용을 참고 바랍니다.

### name 속성은 꼭 챙겨주기

서버와의 통신이 동기식으로 이뤄질 때 보통 `key-value-pair` 형식으로 전달 됩니다.

이 때 입력 양식의 `name` 속성이 `key` 역할을 하게 됩니다.

비동기적으로 쓰일 때도 name 속성은 쓰임새가 많습니다.

가령, 입력값 변동 시 DOM의 name 만으로 그 역할을 판별한다면, 주변의 유사한 이벤트 핸들러를 하나로 통일 할 수 있습니다.

```html
<form onsubmit="handleSubmit(event)">
  <input type="text" name="id" onchange="handleChange(event)" /><br />
  <input type="password" name="pw" onchange="handleChange(event)" /><br />
  <input type="number" name="age" onchange="handleChange(event)" /><br />
  <select name="gender" onchange="handleChange(event)">
    <option value="mal">남성</option>
    <option value="femal">여성</option></select
  ><br />

  <button type="submit">보내기</button>
</form>
```

```js
function handleSubmit(event) {
  event.preventDefault();

  const params = Array.prototype.reduce.call(
    event.target.elements,
    (acc, elem) => {
      if (elem.type !== 'submit') {
        acc[elem.name] = elem.value;
      }
      return acc;
    },
    {}
  );

  console.log(params);
  /*
  완성된 객체로 나올겁니다.
  {
    age: "234",
    gender: "femal",
    id: "theson1",
    pw: "1234"
  }
  */
}

function handleChange(event) {
  const { name, value } = event.target;

  console.log(name, value); // id, theson <-- 이런 형식으로 나옵니다
}
```

또 한 테스트 코드를 작성 할 때 특정 요소를 집어내는 등의 용도로 용이하게 쓸 수 있습니다.

즉, 입력 양식들에 대해 제어를 할 시 그 기준점이 될 수 있습니다.

- [피들러 예제](https://jsfiddle.net/p2zvjyra/)

### 규칙의 의의

입력 양식이 동작되는 영역을 한정 지음으로써 이들에 대한 제어권을 제한 할 수 있습니다.

즉, 너무 많은 곳에서 접근되어 제어되는 양상을 줄일 수 있습니다.

그리고 웹표준에서 지원하는 기본적인 핸들링을 함께 이용함으로써 불필요한 코드를 줄일 수 있습니다.
