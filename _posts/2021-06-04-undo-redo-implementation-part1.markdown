---
layout: post
title: "Undo Redo Implementation Part 1: Command Pattern 학습"
date: 2021-06-04 14:48:00 +09:00
categories: "designpattern"
published: false
---

본 글은 다음 항목을 강하게 참조하고 있습니다.

- Head First Design Pattern의 '6. Encapsulating Invocation: The Command Pattern'
- Design Patterns: Elements of Reusable Object-Oriented Software

[Reference: Head First Design Pattern](https://www.amazon.com/Head-First-Design-Patterns-Brain-Friendly/dp/0596007124)
[Reference: Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8)

## Command Pattern 채택 배경

Undo Redo를 도입함에 있어, 가장 큰 고민거리 중 하나는, 발생한 액션을 어떻게 실행취소 할 것이고, 이걸 어떤 형태로 저장할 것인가에 대한 고민이었다.
무엇보다 본 액션을 수행함에 있어 가장 큰 문제는 아래와 같다.

- 액션은 단순히 직전만 실행 취소 하는 것이 아니라, 특정 단계까지 내려갈 수 있다.
- 여기서 액션은 단일 액션이 아니다. 즉, 생성, 이동, 크기 조절 등 다양한 종류의 액션들이 같은 히스토리에 쌓이게 된다.
- 실행 취소 후, 새로운 액션이 들어가게 되면 기존에 있던 작업들은 모두 사라져야 한다. (일반적인 Undo Redo의 행동 양식)

이런 상황 속에서, 특정 액션을 단일 인터페이스 상에 저장하고, 이후에 다시 복원하거나 취소할 수 있는 매커니즘을 가진 방식으로 구현하는 것이 중요했다.
단일 인터페이스로 구성하기 위해 실행되는 과정 (action)들을 하나의 실행 메소드(execute)로 통일시킬 필요가 있었고, 이를 위해 도입하기로 채택한 Command Pattern (명령 패턴)을 알아보고자 한다.

## Command Pattern 이란?

| Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.
| 요청을 객체로 캡슐화시킴으로써, 서로 다른 요청이나 큐, 혹은 로그 요청을 갖는 클라이언트를 매개변수화하고, 해당 작업들에 대한 실행 취소 기능을 지원한다.

Command Pattern은 행동 패턴(Behavioral Pattern) 중 하나이다. 행동 패턴들은 객체간의 알고리즘과 책임 할당에 관련 있다. 객체나 클래스의 패턴만을 설명할 뿐만 아니라 그들 사이의 커뮤니케이션 패턴도 같이 설명한다. 이런 패턴들은 런타임에 따라가기 어려운 복잡한 제어 흐름을 특정한다. 이 패턴을 통해 제어 흐름에서 벗어나 객체들의 상호 연결 방식에만 집중할 수 있게 해준다.

이 중 하나인 Command Pattern은 메소드 호출을 캡슐화시킨다. 메소드 호출을 캡슐화시킴으로써, 실행을 수행하는 객체는 '어떻게 호출하지?'라는 부분을 생각할 필요가 없게 된다.
예를들어, 아래서 살펴볼 홈 오토메이션의 사례를 살펴보자면, 가전 전체를 관장하는 하나의 리모콘은 제품을 켜고 끄는 버튼, 그리고 이 과정들을 취소할 수 있는 버튼을 가지고 있다.
오해할까봐 먼저 말하지만, Command Pattern의 핵심은 단순히 아래와 같이 제품을 켠다 끈다의 개념이 아니다.

```typescript
class TurnOnSomeLightCommand implements Command {
  ...
  execute(light: Light) {
    light.on();
  }
}
```

위의 예제가 언급하고자 하는 문제는 무엇일까? 바로 실행 단계에서 매개변수로 대상이 될 `light` 객체를 받는다는 것이다.
물론 이런 결정 자체가 이해가 안가는 것은 아니다. 무엇보다 `light`를 켜고 끌 때, '어떤 조명을?' 이란 질문을 던지지 않을 수 없기 때문이다.
Command Pattern은 객체의 생성이 될 때부터 대상까지 지정을 받는다. 즉, 객체 생성 단계에서 '주방 조명'인지 '침실 조명인지'를 모두 갖게 된다.

그리고 이 과정 속에서 얻는 가장 큰 이점은, 이 액션을 실질적으로 실행(invoke)하는 리모콘에서는 단일 인터페이스로 액션들을 실행할 수 있게 된다.

```typescript
class TurnOnPredefinedLightCommand implements Command {
  constructor(target: Light) {
    this.light = target;
  }

  execute() {
    this.light.on();
  }
}

class RemoteControl {
  buttonPresssed(command: Command) {
    command.execute();
  }
}

const light = new Light("bedroom");
const remote = new RemoteControl();
remote.buttonPressed(new TurnOnPredefinedLightCommand(light));
```

여기서 핵심은, 리모콘이 버튼 작동을 해석하여 요청을 보내는 것은 알 수 있지만, 홈 오토메이션이나 욕조를 켜고 끄는 방법에 대해 알 필요는 없다.
가장 안좋은 형태는 아마 아래와 같이 각각의 상황을 리모콘이 모두 알고 있는 것이 아닐까?

```typescript
class RemoteControl {
  buttonPressed(command: Command, target: any) {
    if (command instanceof Light) {
      command.turnLightOn(target);
    } else if (command instanceof Tub) {
      command.turnTubOn(target);
    } else if (command instanceof WiFi) {
      command.turnWifiOn(target);
    }
  }
}
```

### Command interface 도입하기

Light를 켜고 끈다는 과정을 클래스로 만들었을 때 아래와 같이 코드가 나오게 된다.

```java
/**
 * 기본적으로 모두 동일한 형태를 가지기 때문에, Command는 모두 Command interface를 사용한다고 보면 된다.
 */
public class LightOnCommand implements Command {
  Light light;

  /**
   * Constructor에는 해당 Command가 다룰 Control 대상을 명시한다.
   * 여기서 light는 침실의 Light가 될 수도, 거실의 Light가 될 수도 있다.
   */
  public LightOnCommand(Light light) {
    this.light = light;
  }

  public void execute() {
    light.on();
  }
}
```

Command 객체를 사용하기

```java
public class SimpleRemoteControl {
  Command slot;
  public SimpleRemoteControl() {}

  public void setCommand(Command command) {
    slot = command;
  }

  public void buttonPressed() {
    slot.execute();
  }
}

public class RemoteControlTest {
  public static void main(String[] args) {
    SimpleRemoteControl remote = new SimpleRemoteControl();
    Light light = new Light();
    LightOnCommand lightOn = new LightOnCommand(light);

    remote.setCommand(lightOn);
    remote.buttonPressed();
  }
}
```

```java
public class GarageDoorOpenCommand implements Command {
  Garage garage;

  public GarageDoorOpenCommand(Garage garage) {
    this.garage = garage;
  }

  public void execute() {
    this.garage.open();
  }
}
```

Command Pattern: 명령 패턴은 요청을 객체로 캡슐화하므로 다른 요청, 큐 혹은 로그 요청을 가진 객체들을 매개변수화시키고 실행 취소 작업을 가능하게 한다.

명령 객체는 특정 수신자(receiver)에 여러 개의 액션을 바인딩하여 요청을 캡슐화한다. 이를 위해 액션과 수신자를 단 하나의 메소드, `execute()`를 노출하는 객체에 패키징한다. 외부에서 보면, 다른 객체들은 수신자에서 어떠한 액션들이 실행되는지 알지 못한다. 다만 아는 것은 `execute()` 메소드를 실행하면 요청이 실행될 것이라는 점만을 안다.

그렇다면, 이전의 리모콘의 케이스로 다시 돌아와, 언제 어떠한 장치가 제대로 켜지고 꺼지는지 알 수 있는 것일까?
-> 예를 들어, 거실에 조명이 있고, 주방에 조명이 있다면, 그 조명을 켜고 끄는 것에 있어서의 명령 객체는 각각 하나씩 존재하게 된다.

```java
LightOnCommand bedroomLightOn = new LightOnCommand(new Light());
LightOnCommand kitchenLightOn = new LightOnCommand(new Light());
```

꼭 기억해야하는 것이, 요청의 수신자(receiver)는 본인이 캡슐화 되어 있는 명령에 귀속된다. 그렇기에 버튼이 눌리면, 어떤 라이트를 대사으로 하는 것인지 중요하지 않다.

### Invoker: 명령을 할당하기

이제 리모콘의 각각의 슬롯에 명령을 할당하고자 한다. 이 과정을 통해 리모콘은 호출자(invoker)가 된다. 버튼이 눌렸을 때, `execute()` 메소드가 상응하는 명령을 수행할 것이고, 이를 통해 수신자쪽에서 액션이 호출 될 것이다.

```java
public class RemoteControl {
  Command[] onCommands;
  Command[] offCommands;

  public RemoteControl() {
    onCommands = new Command[7];
    offCommands = new Command[7];

    Command noCommand = new NoCommand();
    for (int i = 0; i < 7; i++) {
      onCommand[i] = noCommand;
      offCommand[i] = noCommand;
    }
  }

  public void setCommand(int slot, Command onCommand, Command offCommand) {
    onCommands[slot] = onCommand;
    offCommands[slot] = offCommand;
  }

  public void onButtonPushed(int slot) {
    onCommand[slot].execute();
  }

  public void offButtonPushed(int slot) {
    offCommand[slot].execute();
  }
}
```

### Undo 기능 추가하기

우선, Command interface 내부에 undo가 추가된다면, `undo()`를 interface에 추가해줘야 한다. `execute()`가 수행한 것이 있다면, `undo()`는 그 반대를 한다고 보면 된다.

```java
public interface Command {
  public void execute();
  public void undo();
}
```

이제 `LightOnCommand` 항목에 실제로 `undo()`를 각각 구현해주기만 하면 된다.

```java
public class LightOnCommand implements Command {
  ...

  public void undo() {
    this.light.off();
  }
}
```

이제 리모콘에서 동일하게 `undo()`를 지원해줘야 하기 때문에, 기존 코드들에서 해당 기능을 사용할 수 있게 업데이트 해준다.

```java
public class RemoteControlWithUndo {
  Command[] onCommands;
  Command[] offCommands;
  Command undoCommand;

  public RemoteControlWithUndo() {
    ...

    undoCommand = noCommand;
  }

  /**
   * 이 과정을 통해 최종적으로 눌려져있던 Command가 undoCommand에 저장되게 된다.
   */
  public void onButtonPushed(int slot) {
    undoCommand = onCommands[slot];
  }
  public void offButtonPushed(int slot) {
    undoCommand = offCommands[slot];
  }

  public void undoButtonPushed() {
    undoCommand.undo();
  }
}
```

조금 더 고민해봐야하는 Command의 경우
예를들어, 선풍기의 경우, 이전 값을 기억해야 undo를 할 수 있다는 점이 있다.

이 부분을 좀 더 고민해보자.

```java
public class CeilingFanHighCommand implements Command {
  CeilingFan ceilingFan;
  int prevSpeed;

  public CeilingFanHighCommand(CeilingFan ceilingFan) {
    this.ceilingFan = ceilingFan;
  }

  public void execute() {
    prevSpeed = ceilingFan.getSpeed();
    ceilingFan.high();
  }

  public void undo() {
    if (prevSpeed == CeilingFan.HIGH) {
      ceilingFan.high();
    } else if (prevSpeed == CeilingFan.MEDIUM) {
      ceilingFan.medium();
    } else if (prevSpeed == CeilingFan.LOW) {
      ceilingFan.low();
    } else if (prevSpeed == CeilingFan.OFF) {
      ceilingFan.off();
    }
  }
}

```
