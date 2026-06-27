# Clean Architecture 5부 25장: 계층과 경계

"규칙은 영속성을 가지는 특정한 데이터 구조로 저장한다."

- DB에 테이블형태로 저장을 시킨다는 말인지, 그러니까 DB에서의 엔티티 구조화를 얘기하는 것인가..?

=> 꼭 그런건 아님. DB가 아닐 수도 있음. 질문부터가 세부사항에 결속되었음. 결속되면 안 된다니까? 특정데이터 구조라는 게 DB 테이블로 저장할 수도 있고, JSON 형태가 될 수도 있고, 메모리에 저장되는 형태일 수도.

업무 규칙은 업무 대상의 상태를 기반으로 동작을 하는데,
이 상태는 계속 보존되어야 하므로 영속성을 가진 데이터 구조로 표현됨. 이 과정에서 실제 저장 방식은 세부사항으로, 업무 규칙 자체가 DB 저장 방식에 의존해서는 안 됨.

---

추상 컴포넌트(추상API컴포넌트)와 다형적 인터페이스의 이해

다형적 인터페이스를 통해 제공 : 공통된 약속을 규정해놓고 상황에 맞게 플러그인 해서 쓸 수 있게

```ts
// 추상 API 컴포넌트(Language)
// 다형적 인터페이스 제공

export interface Language {
  greeting(): string;
  translate(text: string): string;
}
```

```ts
// 구체 컴포넌트(English)
import { Language } from "./language";

export class English implements Language {
  greeting(): string {
    return "Hello";
  }

  translate(text: string): string {
    return `[English translation of: ${text}]`;
  }
}
```

```ts
// 구체 컴포넌트(Spanish)
import { Language } from "./language";

export class Spanish implements Language {
  greeting(): string {
    return "Hola";
  }

  translate(text: string): string {
    return `[Spanish translation of: ${text}]`;
  }
}
```

```ts
//클라
import { Language } from "./language";
import { English } from "./english";
import { Spanish } from "./spanish";

function showMessage(language: Language, text: string) {
  console.log(language.greeting());
  console.log(language.translate(text));
}

const english = new English();
const spanish = new Spanish();

showMessage(english, "Good morning");
showMessage(spanish, "Good morning");
```

---

test-suite : 테스트 스위트가 아니고 "테스트 슈트(테스트 케이스 모음)"

---

OverEngineering이 UnderEngineering보다 않 좋음.
-> "경계 구현 비용과 무시했을 때 감수 비용 비교해서 결정"
