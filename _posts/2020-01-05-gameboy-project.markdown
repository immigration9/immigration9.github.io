---
layout: post
title: 'CHIP-8 Project - Week 1'
date: 2020-01-05 01:00:00 +09:00
categories: 'emulation'
published: true
---

## Project 개요

CHIP-8은 70년대의 8비트 마이크로 컴퓨터를 위한 프로그래밍 언어다.
대표적으로 Pong, Space Invaders 같은 유명한 게임들이 있고,
Emulation 이란 필드 자체가 흥미로워 보이기도 하고, C++과 운영체제, 나아가 하드웨어에 대한 이해도를 높이는데 가장 이상적인 것 같아 시도해보는 프로젝트.
목표는 2020년 1분기에 Side project로 완성하는 것이다.

작업을 진행하며 추가적으로 내용들을 더할 예정이다.

### call-by-value

매개변수가 인수값의 복사본을 갖는 동작을 값에 의한 호출 (call-by-value)라고 한다.

lvalue라는 것은 비일시적 객체(nontemporary object)를 나타내는 값. 참조인 변수 혹은 참조를 반환하는 함수를 호출했을 대 결과인 변수가 lvalue이다.ㄴ

### Pointers

x가 객체일 때 &x는 객체의 주소다
p가 객체의 주소일 때 \*p는 객체 자체이다.
