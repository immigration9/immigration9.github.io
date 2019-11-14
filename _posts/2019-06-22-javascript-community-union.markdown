## Electron 시작부터 배포까지

electron-common-ipc: request

```javascript
app.on('ready');
app.on('will-finish-launching');
app.on('second-instance');
```

### 보안

비즈니스 로직을 Node.js로 분리 -> Pkg로 패키징
-> `child_process.spawn` 으로 실행

### Webpack으로 빌드

### Packaging

electron-builder

- 빌드 타겟별 Node.js 네이티브 모듈 처리 ex. gRPC
- Code signing

Windows: NSIS
macOS: Squirrel.Mac
Linux: AppIMage

electron-updater

### End to End Testing

TestCafe
testcafe-browser-provider-electron

### Error Tracking

Sentry

### 개발 경험

Webpack Dev Server + React Hot Loader
mobx-state-tree snapshot으로 HMR에서 상태 유지

Node.js -> `gaze` 모듈로 소스코드 변화를 감지 / Child process를 kill한 후 다시 spawn 하여 리로드

local에서 file 프로토콜로 불러오기 때문에 SSD사용하면 작게 나눠서 로드하는게 최고
-> splitChunk all로 설정

grpc 사용

- protobufjs
- rxjs-grpc

## Electron, React, TS Application 개발하면서 내가 실수한 것들...

Jira로 Sprint 진행
Burnup Chart

문제가 생겼을 때 수정하는 것

### CSS

Flexbox안에서 Scroll 처리문제 (레이아웃 넘치는 현상 등)
flex direction이 중첩되어 들어가면 문제가 생길 수 있음 -> column + column, row + row
Electron -> font-family가 오히려 없는게 낫다
::before / ::after

### MobX

쓴다면 this.state를 쓰지 말자...

Observable Data -> Child Component state 제거

### E2E Testing

Cypress 사용?
eval 사용으로 강제로 크기 조정

## React Hooks + TS + Functional = 아름다움
