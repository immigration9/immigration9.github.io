---
layout: post
title:  "Ubuntu에 환경 세팅하기 (계속 추가)"
date:   2018-12-24 14:00:00 +09:00
categories: "linux"
published: true
---

## 배경설명
환경 세팅에 조건이 여러개 있었다. 맥과 가장 유사해야하고, 파워쉘은 전혀 사용할 줄 모르기 때문에 윈도우는 사용이 어려웠다.. 그래서 최초에 시도한게 WSL (Windows Subsystem for Linux) 였는데, 이게 가장 큰 문제는 아무래도 환경이 분리된거다 보니 무언가 하나를 하려고 하면 두 환경에 모두 세팅해야는 문제가 있었다. 무엇보다... SSH키를 둘 다 만들어서 관리하기가 뭔가 내키질 않았다. 사실 윈도우 쉘을 사용하면 모두 해결되는 문제긴한데, 회사에서 작업은 전부 맥으로 하니 윈도우 쉘을 다시 공부하고 싶진 않았다.

결국 그래서... 우분투를 깔았다.

세상에 진짜 좋다. 물론, 카카오톡 / Bear는 안된다는 단점은 있다.
하지만 생각해보자, 어차피 글은 여기에 남기면 되고, 카카오톡은 없어도 그만이다. Sourcetree도 아쉽긴한데 VSCode에서 지원을 잘 해주니 크게 문제는 없다.

사소한 문제가 몇 가지 있었는데, 그 중 몇가지만 언급해보고자 한다.

1. NPM build 시에 webpack watch 모드가 node_modules 변경 인식 불가
별도의 라이브러리를 만들어 소스를 관리하고 있기 때문에, node_modules를 직접 수정하는 일이 잦다. 근데 문제가, node_modules에 변경되는 사항은 watch 모드가 인식하지 못하는 문제를 발견하였다.
찾아보니, Linux 시스템의 경우 inotify 갯수의 제한 기본값이 8192로 맞춰져있기 때문이였다. (file descriptor와 같다고 보면 될 것 같다)
아래 간단한 코드를 넣고 실행해보니 잘 된다. 땡큐 깃헙!

https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers