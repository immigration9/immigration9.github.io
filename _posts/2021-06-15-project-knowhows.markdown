---
layout: post
title: "Project Knowhows"
date: 2021-06-15 13:00:00 +09:00
categories: "project"
published: false
---

# Github Flow를 이용한 Branch전략

reverting의 최소화/최적화

## Github Flow 흐름

### Branch 생성

- 브랜치 생성시 짧고 명시적인 이름을 사용한다.
  `implement-undo-redo`
- branch prefix에 대하여: `fix/`, `feature/` 정도?

### 변경사항 반영

- Commit message 양식에 관하여 정할 것

### Pull Request 생성

- PR Template 관련 세팅 필요
- 완성되지 않은 것에 대한 피드백이나 조언이 필요할 경우 \[draft\]로 표기하는 방향으로.

### Review Comment 남기기

### Pull Request 머지

### Branch 삭제

- Merge 후 Github에서 삭제

# Component 구조

```
components
hooks
utils
services
pages
layouts
stores
styles
```

# Issue 관리

# versioning 관련

- semver를 따르도록 한다
- https://github.com/JS-DevTools/version-bump-prompt 를 써보는건 어떨까?

# Linting, Local에서 해줘야 하는 일들

- husky + prettier를 사용하여 linting을 강제하자: https://khalilstemmler.com/blogs/tooling/enforcing-husky-precommit-hooks/
- On minimizing context switching: (Move fast with confidence: https://www.youtube.com/watch?v=ikn_dBSski8&t=434s)

```javascript
// lint-staged.config.js
module.exports = {
  "*.{ts,tsx}": [
    "eslint . --fix",
    "prettier --write",
    "git add",
    "jest --bail --findRelatedTests",
  ],
};
```

```javascript
const runYarnLock = "yarn install --frozen-lockfile";

module.exports = {
  hooks: {
    "post-checkout": `if [[ $HUSKY_GIT_PARAMS =~ 1$ ]]; then ${runYarnLock}; fi`,
    "post-merge": runYarnLock,
    "post-rebase": "yarn install",
    "pre-commit": "yarn test && yarn lint-staged",
  },
};
```

- Solving: 'Anyone else having trouble running on from the master branch?'
- HUSKY_GIT_PARAMS ?

# 협업 방향

- 페어프로그래밍
- 테스트 코드 작성
- 코드 리뷰 커버리지 및 방향
- vscode 관련 설정 통일

# 모니터링

- Sentry 도입?
- ErrorBoundary를 적극 도입하고 + Sentry와 같이 모니터링하는 방향도 고민해볼 필요가 있다.

# Git Tips

https://paularmstrong.dev/pages/protips/git
