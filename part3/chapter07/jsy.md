# SRP의 오해

해당 원칙을 이름만 듣는다면 모든 모듈이 하나의 일만 해야 한다는 의미로 받아들임
"단 하나의 일"만하는 것은 함수지 SRP랑 오해하지 말 것

> 함수 : 함수는 반드시 하나의 일만 해야한다
> 이 원칙은 커다란 함수를 작은 함수들로 리팩터링하는 더 저수준에서 사용됨

SRP는 다음과 같다.

> 단일 모듈의 변경의 이유가 하나 오직 하나뿐이어야 한다.

# 소프트웨어의 변화는 언제 이뤄지는가? + SRP의 정의

하나의 모듈은 오직 하나의 사용자 또는 이해관계자에 대해서만 책임져야한다.  
SRP가 말하는 변경의 이유 = 사용자, 이해관계자  
즉, 변경을 요청하는 집단 = ACTOR(액터)

## SRP의 최종 버전

> 하나의 모듈은 하나의 액터에 대해서만 책임져야한다.

Q : 하나의 모듈은 하나의 액터의 요구사항만 받아들여져야한다. 이렇게 이해해도 될까?

# 이 원칙을 위반하는 사례

## 1.우발적 중복

```
class Employee {
  + calculatePay()
  + reportHours()
  + save()
}
```

`Employee` 클래스는 는 CFO, CTO, COO에 의해 사용된다. = 액터가 셋 = SRP 위반

```
calculatePay()는 회계팀에서 기능 정의 -> CFO 보고용
reportHours()는 인사팀에서 기능 정의 -> COO 보고용
save()는 DBA(db 관리자)가 기능 정의 -> CTO 보고용
```

즉 `Employee`라는 단일 클래스에 세 액터가 결합 -> CFO 팀에서 결정한 조치 -> COO팀이 의존하는 무언가에 영향 가능

`calculatePay()`, `reportHours()` 두 메서드가 내부적으로 `regularHours()`라는 함수를 동일한 사용  
 `regularHours()`: 초과 근무시간을 제외한 업무 시간 계산 알고리즘

회계팀에서 사용하는 `calculatePay()` 코드를 수정하면 COO 팀이 사용하는 메서드에도 영향 O  
서로 다른 액터가 의존하는 코드를 너무 가까이 배치했기 때문에 발생, SRP는 서로 다른 액터가 의존하는 코드를 분리하라고 말한다.

### 내 생각

처음에 읽었을 때는 이건 다른 팀(액터)에서도 쓰는 코드를 수정한 개발자 잘못 아닌가?..라고 생각했었음
근데 부제에서도 "우발적 중복"이라고 쓰여있듯이 마침 두 액터에서 사용되는 우연한 중복되는 알고리즘 코드가 있었던것 뿐이고 그 중복된 알고리즘이 아키텍처의 설계 결함으로 인해 버그를 발생시킨것임

아예 분리가 되어 있었다면 이런 일은 없었을 것

## 2.병합

같은 코드가 동시에 수정되면 충돌 일어남

-> 이 문제를 벗어나는 방법 : 서로 다른 액터를 뒷받침하는 코드를 서로 분리

# 해결책

## 1. 데이터와 메서드 분리

아무런 메서드가 없는 간단한 데이터구조인 `EmployeeData` 클래스를 만들어서 세 개의 클래스가 공유하도록 함

```ts
// 엉클 밥이 말한 '아무런 메서드가 없는 간단한 데이터 구조'
export class EmployeeData {
  constructor(
    public id: string,
    public name: string,
    public baseSalary: number, // 기본급
    public hoursWorked: number, // 근무 시간
  ) {}
}

// 1. CFO 팀이 관리하는 클래스 (급여 계산 담당)
export class PayCalculator {
  // 생성자나 메서드로 EmployeeData를 주입받아 사용합니다.
  calculatePay(employee: EmployeeData): number {
    // CFO 팀만의 독자적인 regularHours 계산 규칙을 적용
    const regularHours = this.getCFORegularHours(employee.hoursWorked);
    return regularHours * 10000 + employee.baseSalary;
  }

  private getCFORegularHours(hours: number): number {
    // CFO 팀이 원하는 방식대로 수정한 로직 (COO팀에 영향 없음!)
    return hours > 40 ? 40 : hours;
  }
}

// 2. COO 팀이 관리하는 클래스 (근무 시간 보고 담당)
export class HourReporter {
  reportHours(employee: EmployeeData): string {
    // COO 팀만의 독자적인 regularHours 계산 규칙을 적용
    const regularHours = this.getCOORegularHours(employee.hoursWorked);
    return `${employee.name}님의 이번 주 일반 근무 시간은 ${regularHours}시간입니다.`;
  }

  private getCOORegularHours(hours: number): number {
    // 초기 기존 로직을 그대로 유지 (CFO팀이 바꾸든 말든 노터치)
    return hours;
  }
}

// 3. CEO 팀이 관리하는 클래스 (인사 데이터 저장 담당)
export class EmployeeSaver {
  saveEmployee(employee: EmployeeData): void {
    console.log(`[DB 저장] ${employee.name}의 데이터를 안전하게 저장합니다.`);
    // 실제 DB 저장 로직...
  }
}
```

단점: 이 해결책은 세 가지 클래스를 인스턴스화 해야하고 추적해야 함  
이러한 난관에서 빠져나올 때 흔히 쓰는 기법 = 파사드 패턴

## 2. 가장 중요한 업무 규칙을 데이터와 가깝게 배치하는 방식

가장 중요한 메서드는 기존 Employee 클래스에 그대로 유지, 덜 중요한 나머지 메서드들에 대한 파사드로 사용할 수도 있음

난 이게 더 좋네 무쓸모한 데이터가 아니라 데이터 드리븐한 구조!
좀더 고수준에서 뭘 하는지 알 수 있고 저수준에서는 실제 구현만 되어 있어서 좋은듯

# 결론

SRP는 메서드와 클래스 수준의 원칙

하지만 이보다 상위 수준에서도 다른 형태로 다시 등장 , 컴포넌트, 아키텍처
