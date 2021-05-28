# Coding Conventions: Condition check

조건문 (if, 삼항연산자 등)을 사용할 때의 규칙에 대하여 설명합니다.

## 묵시적 형변환 사용하기

undefined, null, 0, '' 등의 값은 조건문과 사용할 경우에 false 처리됩니다.
따라서, 조건값과 반대인 값을 꼭 사용해야 할 경우가 아니라면 !!의 사용으로 불필요한 연산을 하는 행위를 줄이는 것이 좋습니다.

```tsx
// 삼항연산자 사용 시

interface SomeTextProps {
  message?: string;
}

// no need
function SomeText(props: SomeTextProps) {
  return (
    <p>{!!props.message ? props.message : '기본 메세지'}</p>
  )
}

// suggestion
function SomeText(props: SomeTextProps) {
  return (
    <p>{props.message ? props.message : '기본 메세지'}</p>
  )
}
```

```tsx
// if 문 사용 시

interface UserModel {
  idx: string;
  name: string;
}

const users: UserModel[] = [{
  idx: '1',
  name: 'user1',
},
{
  idx: '2',
  name: 'user2',
}];

function findUserByIdx(users: UserModel[], idx: string) {
  return users.find(user => user.idx === idx);
}

const matchUser = findUserByIdx(users, '1'); // user or undefined

// no need
if (!!matchUser) { 
  // do something with user
}

// suggestion
if (matchUser) {
  // do something with user
}

if (!matchUser) {
  throw new Error('User does not exist');
}
```

## 강제로 형변환 해야하는 경우

```ts
const message = 'some message';
const hasMessage = Boolean(message); // true

const message = '';
const hasMessage = Boolean(message) // false
```