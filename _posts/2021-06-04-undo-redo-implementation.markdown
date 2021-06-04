---
layout: post
title: "Undo Redo Implementation"
date: 2021-06-04 14:48:00 +09:00
categories: "javascript,designpattern"
published: false
---

## Command Pattern

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
