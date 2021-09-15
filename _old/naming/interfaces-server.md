# Naming Conventions: Interfaces - Server Side

서버는 Backend API 를 통해 주고 받는, `UI와는 관계없는` 자료를 다루는 영역 입니다.

이들은 보통 `Repository` 계층에서 사용 됩니다.

- Entity
  - 서버에서 전달 해 준 자료. 응답 자료에 단일로 구성 되거나 **Response Data**의 하위 자료로써 구성될 수 있다.
- Request Parameter
  - 서버에 자료 요청 시 필요한 자료. get/delete 수행 시 사용되는 쿼리 파라미터 뿐만 아니라 put/post 에서 사용되는 request body 도 함께 포함하고 있다.
  - 즉 레포지토리 계층을 사용할 때 이들이 실제 query parameter 인지, request body 인지에 대해서는 관심을 가지지 않도록 만든다.
- Response Data
  - 응답 데이터. 일반적으로 공통으로 사용되는 구조에 정의된다.
  - 따라서 API 호출 결과에 반드시 포함되진 않는다.

| 명칭              | 접미어 | 예시                                    |
| :---------------- | :----- | :-------------------------------------- |
| Entity            | Entity | AuthInfoEntity, CompanyStyleEntity      |
| Request Parameter | Params | UserSignupParams, AddingBoardItemParams |
| Response Data     | Res    | ListItemsRes, ErrorRes, StatusRes       |

## 작성 예시

```ts
// Entity
// 백엔드에서 바로 불러온 raw data 입니다.
interface UserEntity {
  id: number;
  name: string;
  age: number;
  createdAt: string;
  updatedAt: string;
  followCount: number;
  picture: UserPicture;
}

interface UserPicture {
  path: string;
  sizeWidth: number;
  sizeHeight: number;
}

// repository 사용 예시
export function fetchUserInfo(id: number): Promise<UserEntity> {
  // 생략
}
```

```ts
// Request Parameter
// API 요청 시 쓰이는 파라미터 입니다.
interface UserListParams {
  skip: number;
  limit: number;
  keyword?: string;
}

// repository 사용 예시
export function fetchUserList(params: UserListParams): Promise<UserEntity[]> {
  // 생략
}
```

```ts
// Request Parameter
// request body 일 경우의 예시 입니다.
interface UserDetailParams {
  nickName: string;
  name: string;
  age: number;
}

// repository 사용 예시
export function sendUserInfo(params: UserDetailParams): Promise<void> {
  return axios.post({
    url: '/user',
    data: params,
  });
}
```

```ts
// Response Data
// 응답 데이터 유형이 반복되고 내부 엔티티만 변화될 때 쓰입니다.
interface ItemsRes<T> {
  items: T[];
  totalCount: number;
  skip: number;
  limit: number;
}

// repository 사용 예시
export function fetchProductList(
  params: ProductListParams
): ItemsRes<ProductItem> {
  // 생략..
}
```
