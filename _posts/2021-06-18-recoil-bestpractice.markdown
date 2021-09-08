---
layout: post
title: "Recoil Architecture & Best Practices"
date: 2021-06-18 10:00:00 +09:00
categories: "recoil,react"
published: false
---

## Introduction




## Architecture

atom과 selector의 관계

atom의 값에 변화를 주기 위해 사용되는 selector들도 있다.

business logic을 어떻게 분리할 것인가?

대부분의 값들은 작은 단위의 state + 그것에서 derived 되는 성향이 강하다고 느껴짐

atom을 기반으로 Data Fetching을 할 때 어떻게 할 것인지?
- atom을 base로 놔두고 selector에서 참고하도록

atom 수를 최소화하고 최대한 selector를 사용하는 방향으로



# MobX and Recoil

* 동일한 문제점을 해결해준다 -> 효율적인 렌더 + 공유되는 State
* '효율적인 렌더링'이란 뿐에서 React Context와 Redux, 그리고 다른 대부분의 상태관리 라이브러리가 해결하지 못한다.
* Recoil 자체는 observable.box + computed + useObserver (MobX)의 합과 유사.
* Recoil의 강점은 React 바로 위에 만들어졌다는 것이고, 이거 자체가 주는 이점이 명확하다
  * Concurrent Mode 지원이 훨씬 유기적으로 대응이 가능하다
* MobX는 반대로 좀 더 일반적인 목적 성향이 강해서 React가 아닌 프로젝트에서도 사용할 수도 있고, 다른 언어로 포팅되기도 함
  * MobX는 기본적으로 작동하기 위해 React Context를 필요로 하지 않는다
* Recoil과 MobX는 기본적으로 유사: 다양하게 Subscribable한 atom이 있다 (Redux는 항상 one source of truth)
* Redux와 MobX는 기본적으로 작성하는데 있어 굉장히 많은 boilerplate가 있지만, Recoil은 그렇지 않다
  * Concurrent Mode와의 호환은 async 작업에 대해서도 기존에 많은 상태 관리 라이브러리들이 가지고 있는 boilerplate에 대한 고민을 하지 않아도 괜찮다는 것

# Redux and Recoil

* Redux is not to write shorter code, it is used to make your code predictable
* Redux에는 inherent, 그리고 incidental. Indirection도 있고, define action object, dispatch, writing separately와 같은 작업들이 있기에 immutable이 보장되기가 중요하다.
  * 그리고 JS는 이게 default가 아니기에 이걸 만들어줘야 하는 코드가 있었다.
* Redux는 처음 나왔을 때 best in FLUX architecture였다
* 