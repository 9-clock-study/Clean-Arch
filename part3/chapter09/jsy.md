# LSP

리스코프는 하위타입을 아래와 같이 정의

S타입의 객체 o1 각각에 대응하는 T타입 객체 o2가 있고 T 타입을 이용해서 정의한 모든 프로그램 P에서 o2의 자리에 o1을 치환하더라도 P의 행위가 변하지 않는다면 S는 T의 하위 타입이다.

# 상속을 사용하도록 가이드하기

```
class License {
  calcFee()
}

class PersonalLicense extends License { ... }
class BusinessLicense extends License { ... }

```

```ts
class License {
  calcFee() { ... }
}

class PersonalLicense extends License { ... }
class BusinessLicense extends License { ... }

// LSP 덕분에 가능한 것 -> 치환해도 행위가 변하지 않는다는 것이 보장됨 부모 타입에만 의존해도 됨
function billing(license: License) {  // 부모 타입에만 의존
  license.calcFee()
}
```

# 정사각형/직사각형 문제

Rect는 w, h를 독립적으로 관리
square은 side 하나로 관리

사용자는 대화하고 있는 상대가 Rect라고 생각했는데 사실은 square면 코드는 실패하게 됨

```
Rectangle r = new Square()

r.setW(5)
r.setH(2)
assert(r.area() == 10)
```

if문을 사용해서 rect인지 square인지 검사할 수 있으나 이렇게 하면 구체적인 타입에 의존하게 되므로 치환할 수 없게 된다.

# LSP와 아키텍처

LSP는 초기에는 상속을 사용하도록 가이드하는 방법 정도 였으나  
후에 LSP는 인터페이스와 구현체에도 적용되는 더 광범위한 s/w 설계 원칙으로 변모

잘 정의된 인터페이스와 그 인터페이스의 구현체끼리의 상호 치환 가능성에 기대는 사용자들이 늘어났기 때문

# LSP 위배

아까 위에서 설명한 if문 치환 48ln 내용임

# 결론

LSP는 아키텍처 수준까지 확장해라
치환 가능성을 조금이라도 위배하면 시스템 아키텍처가 오염됨
