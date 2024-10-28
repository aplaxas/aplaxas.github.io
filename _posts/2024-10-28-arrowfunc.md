---
layout: single
title: Arrow Functions
---

# 화살표 함수 vs 일반 함수 in TypeScript: 종합 가이드

## 목차

1. [소개](#소개)
2. [구문 비교](#구문-비교)
3. [this 바인딩](#this-바인딩)
4. [렉시컬 스코프](#렉시컬-스코프)
5. [프로토타입과 생성자](#프로토타입과-생성자)
6. [메모리 및 성능 고려사항](#메모리-및-성능-고려사항)
7. [setTimeout과 이벤트 리스너에서의 사용](#settimeout과-이벤트-리스너에서의-사용)
8. [사용 사례](#사용-사례)
9. [결론 및 베스트 프랙티스](#결론-및-베스트-프랙티스)
10. [함수 유형 비교표](#함수-유형-비교표)

## 소개

TypeScript에서 함수를 정의하는 두 가지 주요 방법인 화살표 함수와 일반 함수는 구문과 동작에서 중요한 차이를 보입니다. 이 문서에서는 두 함수 유형의 특성, 차이점, 그리고 적절한 사용 사례를 탐구합니다.

## 구문 비교

### 일반 함수:

```typescript
function regularFunction(a: number, b: number): number {
  return a + b;
}
```

### 화살표 함수:

```typescript
const arrowFunction = (a: number, b: number): number => {
  return a + b;
};
```

화살표 함수는 더 간결한 구문을 제공하며, 특히 단일 표현식을 반환할 때 암시적 반환을 사용할 수 있습니다:

```typescript
const shortArrowFunction = (a: number, b: number): number => a + b;
```

## this 바인딩

`this` 바인딩은 두 함수 유형 간의 가장 중요한 차이점 중 하나입니다.

### 일반 함수:

- 자체적인 `this` 바인딩을 가집니다.
- `this`의 값은 함수가 어떻게 호출되었는지에 따라 결정됩니다.

### 화살표 함수:

- 자체 `this`를 바인딩하지 않습니다.
- 렉시컬 스코프의 `this`를 사용합니다 (즉, 함수가 정의된 곳의 `this`를 사용).

예제:

```typescript
class Example {
  name: string = "Example";

  regularMethod() {
    console.log("Regular:", this.name);
  }

  arrowMethod = () => {
    console.log("Arrow:", this.name);
  };

  demonstrate() {
    const standalone = this.regularMethod;
    standalone(); // 출력: "Regular: undefined"

    const standaloneArrow = this.arrowMethod;
    standaloneArrow(); // 출력: "Arrow: Example"
  }
}

const ex = new Example();
ex.demonstrate();
```

## 렉시컬 스코프

화살표 함수는 렉시컬 스코프를 사용하여 `this`를 결정합니다. 이는 함수가 정의된 위치의 `this`를 사용한다는 의미입니다.

```typescript
class LexicalScopeExample {
  name: string = "LexicalScope";

  delayedRegular() {
    setTimeout(function () {
      console.log("Regular delayed:", this.name); // undefined
    }, 100);
  }

  delayedArrow() {
    setTimeout(() => {
      console.log("Arrow delayed:", this.name); // "LexicalScope"
    }, 100);
  }
}

const lexExample = new LexicalScopeExample();
lexExample.delayedRegular(); // 출력: "Regular delayed: undefined"
lexExample.delayedArrow(); // 출력: "Arrow delayed: LexicalScope"
```

## 프로토타입과 생성자

### 일반 함수:

- `new` 키워드와 함께 생성자로 사용할 수 있습니다.
- 프로토타입에 메서드를 추가할 수 있습니다.

### 화살표 함수:

- 생성자로 사용할 수 없습니다.
- 프로토타입 체인에 참여하지 않습니다.

예제:

```typescript
function RegularConstructor(this: any, name: string) {
  this.name = name;
}

RegularConstructor.prototype.greet = function () {
  console.log(`Hello, I'm ${this.name}`);
};

const regularInstance = new (RegularConstructor as any)("Regular");
regularInstance.greet(); // 출력: "Hello, I'm Regular"

const ArrowConstructor = (name: string) => {
  this.name = name; // 오류: 화살표 함수에서 'this'는 전역 객체를 가리킴
};

// const arrowInstance = new ArrowConstructor("Arrow"); // 오류: ArrowConstructor는 생성자로 사용할 수 없음
```

## 메모리 및 성능 고려사항

### 일반 함수:

- 클래스 메서드로 사용될 때 프로토타입에 정의됩니다.
- 모든 인스턴스가 같은 함수를 공유하므로 메모리 효율적입니다.

### 화살표 함수:

- 클래스 메서드로 사용될 때 각 인스턴스마다 새로운 함수가 생성됩니다.
- 인스턴스별로 독립적인 함수가 생성되므로 메모리 사용량이 더 많을 수 있습니다.

## setTimeout과 이벤트 리스너에서의 사용

`setTimeout`과 이벤트 리스너에서 함수를 사용할 때, `this` 바인딩의 차이가 중요해집니다:

```typescript
class TimeoutExample {
  name: string = "Timeout";

  regularTimeout() {
    setTimeout(function () {
      console.log("Regular timeout:", this.name); // undefined
    }, 100);
  }

  arrowTimeout() {
    setTimeout(() => {
      console.log("Arrow timeout:", this.name); // "Timeout"
    }, 100);
  }

  boundRegularTimeout() {
    setTimeout(
      function () {
        console.log("Bound regular timeout:", this.name); // "Timeout"
      }.bind(this),
      100,
    );
  }
}

const timeoutExample = new TimeoutExample();
timeoutExample.regularTimeout();
timeoutExample.arrowTimeout();
timeoutExample.boundRegularTimeout();
```

## 사용 사례

### 일반 함수에 적합한 경우:

- 객체의 메서드로 사용될 때 (객체의 `this`를 참조해야 하는 경우)
- 생성자 함수로 사용될 때
- 프로토타입 메서드로 사용될 때

### 화살표 함수에 적합한 경우:

- 콜백 함수로 사용될 때 (예: `setTimeout`, 이벤트 리스너)
- 함수형 프로그래밍 패턴에서 (map, reduce 등)
- 클래스 필드에서 메서드를 정의할 때 (`this` 바인딩 문제를 피하기 위해)

## 결론 및 베스트 프랙티스

1. 객체 메서드나 프로토타입 메서드를 정의할 때는 일반 함수를 사용하세요.
2. 콜백이나 짧은 함수식에는 화살표 함수를 사용하세요.
3. 클래스 필드에서 메서드를 정의할 때 `this` 바인딩 문제를 피하려면 화살표 함수를 사용하세요.
4. 생성자 함수가 필요한 경우 일반 함수를 사용하세요.
5. 성능이 중요한 경우, 많은 인스턴스를 생성하는 클래스에서는 프로토타입 메서드(일반 함수)를 선호하세요.
6. `this`의 값이 동적으로 결정되어야 하는 경우 일반 함수를 사용하세요.
7. 렉시컬 스코프의 `this`를 캡처해야 하는 경우 화살표 함수를 사용하세요.

각 함수 유형의 특성을 이해하고 상황에 맞게 적절한 유형을 선택하는 것이 중요합니다. TypeScript의 타입 체킹 기능을 활용하여 `this` 바인딩 관련 오류를 조기에 발견하고 수정할 수 있습니다.

## 함수 유형 비교표

| 특성             | 일반 함수                       | 화살표 함수                         |
| ---------------- | ------------------------------- | ----------------------------------- |
| 구문             | `function name(params) { ... }` | `(params) => { ... }`               |
| this 바인딩      | 동적 바인딩 (호출 시점에 결정)  | 렉시컬 바인딩 (정의 시점에 결정)    |
| 생성자 사용      | 가능                            | 불가능                              |
| 프로토타입       | 있음                            | 없음                                |
| arguments 객체   | 사용 가능                       | 사용 불가 (대신 rest 파라미터 사용) |
| 메서드로 사용 시 | 객체의 메서드로 적합            | 객체의 메서드로 사용 시 주의 필요   |
| 클래스 내부      | 프로토타입 메서드로 정의        | 인스턴스 메서드로 정의              |
| 호이스팅         | 함수 선언식의 경우 호이스팅됨   | 호이스팅되지 않음                   |
| 암시적 반환      | 불가능                          | 가능 (단일 표현식인 경우)           |
| 메모리 사용      | 프로토타입 체인 사용으로 효율적 | 각 인스턴스마다 새 함수 생성        |
| 콜백 함수로 사용 | this 바인딩 주의 필요           | this 바인딩 문제 없음               |
| 가독성           | 전통적인 형태로 익숙함          | 간결한 구문으로 짧은 함수에 적합    |
| 유연성           | this를 동적으로 바인딩 가능     | this가 고정되어 있음                |

이 표는 화살표 함수와 일반 함수의 주요 차이점을 간략하게 요약합니다. 각 특성에 따라 어떤 함수 유형이 더 적합한지 빠르게 판단할 수 있습니다. 개발 상황과 요구사항에 따라 적절한 함수 유형을 선택하는 데 이 비교표가 도움이 될 것입니다.
