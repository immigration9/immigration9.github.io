---
layout: post
title: 'Linux Kernel Basics'
date: 2019-09-02 18:05:00 +09:00
categories: 'linux'
published: false
---

- 본 내용은 Jason Wertz의 Kernel Basics 영상을 보고 정리한 내용입니다: [Kernel Basics](https://www.youtube.com/watch?v=rTcnTOXf_jM)

## 커널이란 무엇인가?

커널은 소프트웨어로부터 들어오는 입출력 요청을 받아 CPU와 여타 컴퓨터의 부품들을 위해 데이터 처리 명령으로 변환하는 작업을 수행하는 컴퓨터 프로그램이다.
하드웨어 컨택하는 가장 밑단에 있는 소프트웨어라고 보면 된다.
Ubuntu의 경우 `/boot` 경로 밑에 있는 `vmlinuz`가 커널 파일이다.
`/lib/modules` 경로 안에는 실제 커널 파일들이 위치해있다.

```shell
uname -a
Linux ... 5.0.0-25-generic
```

`lsmod` 명령어를 통해 설치된 커널 모듈들을 확인할 수 있다.

```shell
sudo apt-get install libncurses5-dev bison flex
make menuconfig

make all
make modules_install
make install # kernel 설정
```

이후에는 Bootloader에 커널을 인식하도록 해줘야 한다.
