---
layout: post
title: "왜 React Context는 '상태 관리' 도구가 아닌가? (그리고 왜 Redux를 대체하지 못하는가)"
date: 2021-06-19 11:50:00 +09:00
categories: "redux,react"
published: false
---

## Article 소개

본 글은 Redux의 maintainer인 Mark "acemarke" Erikson의 Blogged Answers 시리즈 중 "Why React Context is Not a 'State Management' Tool (and Why It Doesn't Replace Redux)"를 번역한 글입니다.

[원문: Why React Context is Not a 'State Management' Tool (and Why It Doesn't Replace Redux)](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/)

많은 분들이 알고 계시지만, 굉장히 자주 잘못 사용되는 용어다보니 한 번 쯤은 이런 것에 대해 자세히 설명해주는 글이 있으면 좋겠다 싶었는데, 아무래도 Redux maintainer가 직접 디테일하게 관련 내용을 상세하게 작성해준 내용이 있어 Context API 뿐만 아니라, Redux를 사용함에 있어서도 좀 더 클리어한 이유를 들을 수 있었습니다.

다소 긴 글이지만, 항상 타인에게 설명하기엔 애매한 포인트들이 많았던 이 둘의 관계에 대해 좀 더 쉽게 이해하는데 있어 도우밍 되었으면 하는 바입니다.

Also, special thanks to Erikson for letting me translate this wonderful article!

## Introduction

"Context vs Redux"는 현재의 React Context API가 배포된 뒤로 React 커뮤니티상에서 가장 많이 다뤄진 토론 주제 중 하나이다. 슬프게도, 여기서 대부분의 "토론"은 두 개의 서로 다른 도구의 목적과 유즈 케이스에 대한 혼동에서 온다. 인터넷 상에서 나는 Context와 Redux를 다루는 질문에 대해 수 백번 답변하였지만([Redux, 아직 죽지 않았어!](https://blog.isquaredsoftware.com/2018/03/redux-not-dead-yet/), [React, Redux, 그리고 Context의 작동방식](https://blog.isquaredsoftware.com/2020/01/blogged-answers-react-redux-and-context-behavior/), [React의 렌더링 방식에 대한 (거의 완벽) 가이드](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/), [언제 Redux를 사용해야(혹은 사용하지 않아야) 하는지](https://changelog.com/posts/when-and-when-not-to-reach-for-redux)와 같은 글에서 특히), 혼동은 갈수록 심각해지고 있다.

이 주제에 대해 너무나도 많은 질문이 있어, 이 글을 통해 모든 질문들에 대한 확정 답변을 전달하고자 한다. Context와 Redux가 정확히 어떤 것인지 설명하고, 어떻게 사용되어야 하며, 어떻게 다르고, 그리고 언제 사용해야하는지 설명하겠다.

### TL;DR

#### Context와 Redux는 같은 것인가?

아니다. 둘은 다른 일을 하는 다른 도구이며, 다른 목적을 위해 사용한다.

#### Context는 '상태 관리(State Management)' 도구인가?

아니다. Context는 의존성 주입(Dependency Injection)의 형태라 볼 수 있다. 하나의 전송 장치(transport mechanism)로 그 어느것도 '관리'하지 않는다. 모든 '상태 관리'는 주로 `useState/useReducer`를 통한 너의 코드를 통해 이뤄진다.

#### Context와 useReducer는 Redux의 대체품인가?

아니다. 비슷한 부분과 겹치는 부분도 있지만, 서로의 가능성에 있어 굉장히 큰 차이점이 있다.

#### 언제 Context를 사용해야할까?

React 컴포넌트 트리 상에서 상위부터 하위 컴포넌트로 props를 통해 값을 내려주지 않고도 항상 접근이 가능한 값을 가지려 할 때 사용한다.

#### 언제 Context와 useReducer를 사용해야할까?

애플리케이션의 특정 부분에서 React component에 대한 다소 복잡한 상태 관리가 필요한 경우.

#### 언제 Redux를 대신 사용해야할까?

Redux는 아래와 같은 케이스에 있어 특히 유용하다

- 애플리케이션의 여러 곳에서 많은 양의 애플리케이션 상태를 필요로 할 경우
- 애플리케이션 상태가 시간이 지남에 따라 주기적으로 업데이트 되는 경우
- 상태를 업데이트하는 로직이 복잡한 경우
- 애플리케이션이 중간 - 큰 규모의 코드베이스를 가지고 있으며, 많은 사람들이 개발에 참여하고 있는 경우
- 애플리케이션 내부의 상태가 언제, 왜, 그리고 어떻게 업데이트되는지 이해하고 싶고, 시간의 변화에 따른 상태의 변화에 대해 시각화를 하고 싶을 때
- 사이드이펙트, 지속성, 데이터 시리얼라이징(side effects, persistence, and data serialization) 등의 강력한 기능들이 필요할 때

## 목차

## Context와 Redux 이해하기
