# 클린 아키텍처

시스템 아키텍처와 관련된 여러 아이디어들

- hexagonal Architecture
- DCI(Data, Context and Interaction)
- BCE(Boundary-Control-Entity)

모두 세부적인 면에서는 다소 차이가 있지만 이들의 목표는 Separation of concerns임

또한 아키텍처들은 모두 다음과 같은 특징을 지니도록 만든다.

- 프레임워크 독립성
- 테스트 용이성
- UI 독립성
- DB 독립성
- 모든 외부 에이전시에 대한 독립성 : 실제로 업무 규칙은 외부 세계와의인터페이스에 대해 전혀 알지 못함

**아래서부터는 모두 p215에 있는 그림-22.1을 토대로 설명**

## 의존성 규칙

동심원에서 안으로 들어갈수록 고수준에 해당, 이 동심원을 유지하는 규칙은 의존성 규칙

"소스 코드 의존성은 반드시 안쪽으로, 고수준의 정책을 향해햐 한다."

외부의 원에 선언된 데이터 형식도 내부의 원에서 절대로 사용해서는 안된다.

### 근데 내 생각은 좀 다름

소규모 프로젝트에서는 이렇게 내부에서 외부 의존을 하지 않을수가 없는게 특히! 테이블 이름 때문에

Post라는 테이블을 만들면 해당 Post(즉 게시글)를 내부 유스케이스 부분에서 무조건 쓰일텐데 이 부분을 어떻게 다 역전시켜?
그러면 거의 모든 모듈들 마다 DTO와 관련된 인터페이스를 둬야하는데 너무 번거롭지 않나?.. 이정도는 괜찮은건가?

내가 지금 찾은 해결책은 아래 코드와 같음

```ts
export const users = table(
  "users",
  {
    id: t.integer().primaryKey().generatedAlwaysAsIdentity(),
    firstName: t.varchar("first_name", { length: 256 }),
    lastName: t.varchar("last_name", { length: 256 }),
    email: t.varchar().notNull(),
    invitee: t.integer().references((): AnyPgColumn => users.id),
    role: rolesEnum().default("guest"),
  },
  (table) => [
    t.uniqueIndex("email_idx").on(table.email)
  ]
);
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;

이렇게 테이블을 만들고 테이블 기반에서 타입을 추출한단 말이지

이 때 해당 타입을 곧바로 다른 유스케이스에 쓰는게 아니라

유스케이스에서는 이렇게 User를 만듬
interface UserSummary {
  firstName: string;
  lastName: string;
  email: string;
  invitee: number;
  role: 'guset' | 'admin'
}

만든 유저를 유스케이스 업무규칙 쪽에서 사용하고

실제 createUser 로직에서 db의 NewUser타입으로 실제 객체를 비교하면됨
const user = {
  firstName: 'Hunter'
  lastName: 'Joe'
  email: 'none'
  invitee: 11
  role: 'guest'
} satisfies NewUser

이렇게 코드를 짜면 실제 유스케이스에서는 테이블 타입을모르고 내가 선언한 타입으로만 진행하니 테이블이 바뀌어도 영향 없음
하지만 테이블이 바뀐다면? 마지막 satisfies 부분에서 내가 만든 객체가 NewUser인지 컴파일 타임에 검사 이러면 마지막에 필드들만 검사할테니깐 결합도를 확실히 낮출 수 있음

아니면 아예 mapper를 둬서 변환 작업을 거친다거나 할 수 있음
```

## 경계를 횡단하는 데이터는 어떤 모습인가.

경계를 가로지르는 데이터는 흔히 간단한 데이터 구조로 이루어져 있다. 기본적인 구조체, 간단한 DTO 등 원하는 대로 고를 수 있다.
중요한 점은 격리되어 있는 간단한 데이터 구조가 경계를 가로질러 전달된다는 사실이다. 엔티티 객체나 데이터베이스의 행을 전달하는 일은 원치 않음

예를 들어 많은 데이터베이스 프레임워크는 쿼리에 대한 응답으로 사용하기 편한 데이터 포맷을 사용하는데 이러한 포맷은 row 구조인 경우가 많다. 그렇기에 이 row 구조가 경계를 넘어 내부로 그대로 전달되는 것을 원치 않는다. 이렇게 되면 의존성 규칙을 위배 + 내부의 원에서 외부 원의 무언가를 알아야 하기 때문
**따라서 경계를 가로질러 데이터를 전달할 때 데이터는 항상 내부의 원에서 사용하기에 가장편리한 상태를 가질 것**
