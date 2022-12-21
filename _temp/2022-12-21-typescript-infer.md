# Typescript Conditional Type && infer keyword

## Reference

[Typescript Official Docs](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
[Matt Pocock - infer is easier than you think](https://www.youtube.com/watch?v=hLZXJTm7TEk)

## Conditional Types (조건부 타입)이란?

유용한 프로그램의 중심에는 언제나 입력을 기반으로 한 결정들을 만들어야 한다. 조건부 타입은 입력값과 출력값의 관계를 설명하는데 도움을 준다.

```typescript
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string; // number type이 된다

type Example2 = RegExp extends Animal ? number : string; // string type이 된다
```

조건부 타입의 진정한 진가는 제네릭과 조합되었을 때 빛이 난다. 오버로딩을 추가할 때마다 처리해줘야 하는 타입의 양이 기하급수적으로 늘어나기 때문이다.

```typescript
type StringOrNumber<T extends number | string> = T extends number
  ? SomeNumberType
  : SomeStringType;

function createStringOrNumber<T extends number | string>(): StringOrNumber<T>;

const numValue = createStringOrNumber(12345); // SomeNumberType
const strValue = createStringOrNumber("hello"); // SomeStringType
```

### 조건부 타입의 제약사항

조건부 타입 체킹은 종종 새로운 정보를 제공해줄 수 있다. 타입 가드를 통해 특정한 타입으로 좁혀나갈 수 있듯이 조건부 타입의 true에 해당하는 브랜치는 우리가 체크하는 타입을 기준으로 제네릭을 제한할 수 있다.

예를들어 아래와 같은 코드는 오류를 발생시킨다.

```typescript
type MessageOf<T> = T["message"];
```

위 코드에서 에러가 발생하는 것은 `T`라는 제네릭 안에 실제로 `message`라는 속성이 존재하는지 알 수 없기 때문이다. 이 문제를 해결하고자 한다면 `T extends`를 통해 해당 제네릭 안에는 `message` 속성이 존재함을 명시해줘야 한다.

```typescript
type MessageOf<T extends { message: unknown }> = T["message"];
```

하지만, 만약에 `MessageOf`가 아무 타입이나 받고, 해당되지 않을 경우 `never`와 같은 속성으로 fallback하게 하고 싶다면? 아래와 같이 조건부 타입을 이용하여 제약사항을 두면 된다.

```typescript
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

type EmailMessageContents = MessageOf<{ message: string }>; // string type
type SomeOtherMessageContents = MessageOf<"hello">; // never type
```

## 조건부 타입 내에서 추론

이제 다시 본 주제로 돌아왔다. 조건부 타입은 `infer` 키워드를 통해 우리가 true 분기에서 사용하는 타입을 추론할 수 있도록 해준다. 예를들어 `Flatten`과 같은 타입을 작성한다고 가정하였을 때, 수동으로 인덱스 접근 타입(여기선 `number`)를 이용하여 가져오는 것이 아니라, 추론된 요소 값을 가져올 수 있다.

```typescript
type Flatten<Type> = Type extends Array<infer Item> ? Item : never;
```

위 예제에서 `infer` 키워드를 사용하여 true 분기에서 요소의 타입 `T`를 어떻게 가져올지 지정해주는 대신, 선언적으로 `Item`이라는 새로운 제네릭 타입을 소개한 것을 볼 수 있다.

즉 요약하자면 `infer` 키워드는 조건부 타입 내에서 타입을 생성하는 방법이다.

### ReturnType

놀랍게도 `infer` 키워드를 이용하여 조건부 타입 내에서 추론을 한 가장 대표적인 사례가 바로 많이 사용되는 `ReturnType`이다. 실제로 코드 내부를 살펴보면, `ReturnType`은 `infer` 키워드를 이용하여 전달된 함수의 결과값을 가져오는 타입의 alias임을 확인할 수 있다.

```typescript
type ReturnType<T extends (...args: any) => any> = T extends (
  ...args: any
) => infer R
  ? R
  : any;
```

하나의 정규 표현식과 같다고 생각하는 것이 이해해 도움이 된다.
즉 매칭이 되면 추론된 타입을 사용할 수 있고, 그렇지 않으면 사용하지 못하는 것이다.

설명하자면, `ReturnType`은 제네릭으로 `(...args: any) => any` 함수형 타입을 상속받는 타입 `T`를 갖는데, 이 타입 `T`가 `(...args: any) => ` 의 형태를 충족시키면 true 분기에서 사용할 수 있도록 출력 값을 `infer` 해줄 수 있다.

여기 들어가는 `R`은 아무 이름이나 사용할 수 있다.

### infer 타입의 패턴 매칭

`infer` 키워드는 기본적으로 전달되는 패턴을 매칭하고자 하기 때문에 비단 함수형 타입에만 적용되는 것이 아니다. 아래와 같이 객체 타입의 특정 프로퍼티 값을 `infer` 해줄 수도 있다.

```typescript
type GetFromObject<T> = T extends {
  someProp: infer P;
}
  ? P
  : never;

type Result = GetFromObject<{ someProp: boolean }>; // bolean 타입이 된다
type Wrong = GetFromObject<{ someOtherProp: boolean }>; // 매칭되지 않기에 never 타입이 된다
```
