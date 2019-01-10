---
layout: post
title:  "Proxy Pattern"
date:   2019-01-11 01:00:00 +09:00
categories: "design-patterns"
published: true
---

## 의도
다른 객체에 대한 접근을 제어하기 위한 대리자의 역할을 하는 객체.
-> 객체에 대한 접근을 제어하는 한 가지 이유는 실제로 그 객체를 사용할 수 있을 때까지 객체 생성과 초기화에 들어가는 비용 및 시간을 물지 않겠다는 것.

## 종류
1. 가상 프록시: 생성에 코스트가 많이 드는 작업을 수행. 요청시에만 해당 객체를 생성.
2. 원격지 프록시: 서로 다른 주소 공간에 존재하는 객체를 가리키는 객체.
3. 보호용 프록시: 원래 객체에 대한 실제 접근을 제어
4. 스마트 프록시: 실제 객체에 접근시 추가적인 행동을 수행.
- 객체의 레퍼런스에 대한 숫자 카운팅 (래퍼런스 없을 시 객체 할당 릴리즈)
- 최초 레퍼런스시 객체를 메모리에 로드
- 객체 수정시 다른 클라이언트가 객체를 수정하지 못하도록 잠금

## 가상 프록시 (Virtual Proxy)
* 인터페이스: Image.java
```java
package proxy.imageProxy;

public interface Image {
    void display();
}
```

* 구현체: RealImage.java
```java
package proxy.imageProxy;

public class RealImage implements Image {
    private String fileName;

    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk(fileName);
    }

    @Override
    public void display() {
        System.out.println("Displaying: " + this.fileName);
    }

    private void loadFromDisk(String fileName) {
        System.out.println("Loading: " + fileName);
    }
}
```

* 프록시: ProxyImage.java
```java
package proxy.imageProxy;

public class ProxyImage implements Image {
    private RealImage realImage;
    private String fileName;

    public ProxyImage(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}
```

* 메인 클래스: ProxyPatternDemo.java
```java
package proxy.imageProxy;

public class ProxyPatternDemo {
    public static void main(String[] args) {
        Image image = new ProxyImage("test_test_big.png");
        System.out.println("At this stage, the image is not created");

        image.display();
        System.out.println("==============================");

        image.display();
    }
}
```

이 방식의 이점은, 실질적으로 `display` 함수가 호출되기 전까진, 이미지 파일이 로드 되지 않는다는 것이다. 이렇게 함으로써 이미지와 같은 고비용 객체를 필요시에만 로드할 수 있다.
* 코드 출처: TutorialsPoint, https://www.tutorialspoint.com/design_pattern/proxy_pattern.htm

## 보호용 프록시 (Protection Proxy)
* 인터페이스: Internet.java
```java
package proxy.protectionProxy;

public interface Internet {
    public void connectTo(String hostname) throws Exception;
}
```

* 구현체: RealInternet.java
```java
package proxy.protectionProxy;

public class RealInternet implements Internet{
    @Override
    public void connectTo(String hostname) throws Exception {
        System.out.println("Connecting to: " + hostname);
    }
}
```

* 프록시 구현체: ProxyInternet.java
```java
package proxy.protectionProxy;

import java.util.ArrayList;
import java.util.List;

public class ProxyInternet implements Internet {
    private Internet internet = new RealInternet();
    private static List<String> bannedSites;

    static {
        bannedSites = new ArrayList<String>();
        bannedSites.add("aaa.com");
        bannedSites.add("bbb.com");
        bannedSites.add("ccc.com");
        bannedSites.add("ddd.com");
        bannedSites.add("eee.com");
    }

    @Override
    public void connectTo(String hostname) throws Exception {
        if (bannedSites.contains(hostname.toLowerCase())) {
            throw new Exception("Access Denied");
        }
        internet.connectTo(hostname);
    }
}
```

* 클라이언트: Client.java
```java
package proxy.protectionProxy;

public class Client {
    public static void main (String[] args) {
        Internet internet = new ProxyInternet();
        try {
            internet.connectTo("immigration9.whatap.io");
            internet.connectTo("www.google.com");
            internet.connectTo("aaa.com");
            internet.connectTo("bbb.com");
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

이 방법을 사용할 경우, 해당 객체 내에 있는 부분을 권한에 따라 제한할 수 있다. 예를들어, ERP 소프트웨어라고 할 때, ERP 로직 자체는 유저 권한과 관련 없이 생성하고, 권한 관리를 Proxy쪽으로 추출하여, Admin과 아닌 유저를 기반으로 사용 여부를 결정할 수도 있다.

이렇게 될 경우, 객체별로 접근 제어 권한이 다를 때 사용에 유용할 수 있다.
* 코드 출처: Geeks for Geeks, https://www.geeksforgeeks.org/proxy-design-pattern/