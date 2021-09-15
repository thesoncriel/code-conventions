# Git: Commit Message

## Commit

기본적으로 `Conventional Commits` 규칙을 따릅니다.

### Commit Message - Title

**필수(Required)** 입니다.

커밋 메시지 중 제목은 다음과 같은 규칙을 따릅니다.

```
[commit type] : [subject]
```

#### Commit Type

커밋 타입은 해당 작업의 성격에 따라 아래와 같이 정의됩니다.

- feat: 새로운 기능 추가
- fix: 버그 수정
- docs: markdown 같은 문서 작성, jsdoc 주석을 통한 라이브러리 관리
- style: eslint 나 style-lint 같은 코드 포맷팅, prettier 를 통한 변경, code convention 규칙 누락에 따른 수정 등
- refactor: 코드 리팩토링
- test: jest 를 통한 단위 테스트, storybook 같은 컴포넌트 테스트, 혹은 cypress 를 이용한 e2e 테스트
- chore: 빌드나 CI, github action, package 수정 등 상기 업급된 외의 자질구레한 모든 것들

#### Subject

제목은 한번에 50자를 넘기지 않습니다.

만약 넘기게 된다면 아래 `Body`를 참고하여 장문을 작성하되, 제목은 간결하게 바꿔 줍니다.

#### 작성 예시

아래는 어디까지나 예시이며 커밋 타입은 반드시 지키되 제목 부분은 어떤 업무를 진행 했는지만 명확히 적어둡니다.

```
feat: 로그인 화면에 email 유효성 검증 기능 추가

fix: 회원 가입 시 발생된 토큰 오류 수정

refactor: 상품 검색 목록의 중복된 코드 제거

docs: README.md 의 빌드 내용 보완

test: 주문서에 cypress 를 이용한 e2e 테스트 코드 추가

chore: package.json 내 버전업

chore: tsconfig 및 eslint 내용 업데이트
```

### Commit Message - Body

**선택사항(Optional)** 입니다.

커밋 메시지가 많을 경우 부가적인 설명을 위해 적습니다.

자유롭게 적되 내용이 너무 많아질 경우 PR에서 별도로 설명하거나 commit 을 나눠서 가급적 메시지를 간결히 유지하도록 합니다.

### Commit Message - Footer

**선택사항(Optional)** 입니다.

기본적으로 연관되어 참고 할 이슈 ID를 넣는 공간 입니다.

가령, 오류가나 hotfix 를 했는데, 문제가 발생된 이슈ID를 알고 있어 이를 참고하고자 할 때 추가 할 수 있습니다. (어디까지나 선택사항)

아래는 작성 가능한 타입 입니다.

- Resolves: 현재 커밋과 연관된 다른 이슈 ID. 푸터 추가 시 기본적으로 작성되어야 할 항목 입니다.
- See also: 또 다른 이슈 ID 모음. Resolves 만으로 모자라거나 부차적으로 여러개를 참고할 필요가 있다면 달아줍니다.

#### 전체 작성 예시

아래는 제목과 바디, 푸터가 함께 들어간 예시 입니다.

다소 과장된 내용이 있으므로 어디까지나 작성 방법에 대해 참고만 바랍니다.

```
// 제목
feat: 회원탈퇴 기능 추가

// 본문 - 문장 예시
서비스를 개발 하였으나 회원탈퇴가 기존엔 불가하여 추가하게 된 기능 입니다.

// 본문 - 목록 예시
- 회원탈퇴 사유를 받는 페이지 개발
- 해당 페이지의 라우팅 적용
- 전용 non-standard 컴포넌트 추가

// 푸터
Resolves: #456 <- 메인 이슈 ID.
See also: #6678, #1123 <- 추가적으로 달아준 이슈 ID.
```

## 참고

- https://www.conventionalcommits.org/ko/v1.0.0/
- https://doublesprogramming.tistory.com/256
