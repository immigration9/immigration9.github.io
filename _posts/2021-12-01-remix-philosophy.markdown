---
layout: post
title: "Remix Philosophy"
date: 2021-12-01 00:50:00 +09:00
categories: "react,remix"
published: true
---

아래 글의 원문은 [Remix Philosophy](https://remix.run/docs/en/v1/guides/philosophy) 입니다.

# Philosophy

우리는 정말 많은 종류의 서로 다른 웹사이트를 만들었다. 몇 가지만 꼽아보자면 카드사를 위한 정적 사이트, 소셜 미디어 플랫폼, 학습 관리 시스템, CMS, 그리고 이커머스 등이 있다. 동시에 [React Training](https://reacttraining.com/)을 통해 수백개의 개발팀을 교육시켰다. 우리 개개인의 개 경험과 클라이언트들이 맡긴 제품들을 기반으로, 웹 프로젝트의 프론트엔드와 백엔드 양쪽 모두가 가지고 있는 동적인 특징을 잘 다룰 수 있도록 Remix를 만들었다.

Remix의 철학은 4개로 요약할 수 있다.

1. Server/Client 모델을 차용하며, 소스코드로부터 컨텐츠와 데이터 분리를 한다.
2. Browsers, HTTP, 그리고 HTML로 대표되는 웹의 기초를 기반으로 (반하는 것이 아닌) 작업한다. 이들은 항상 좋았고, 최근 몇 년 사이에는 정말 좋아졌다.
3. JavaScript를 통해 Browser 행동양식을 에뮬레이트하여 유저 경험을 증대시킨다.
4. 기반에 깔린 기술들을 지나치게 추상화하지 않는다.

## 서버/클라이언트 모델 (Server/Client Model)

당신의 서버를 빠르게 할 수 있지만, 사용하는 유저의 네트워크는 빠르게 할 수 없다.

오늘 날의 웹 인프라 상태에서 서버를 빠르게 하기 위해 정적 파일을 굳이 만들 필요까진 없다. 지금 보고 있는 이 사이트는 정말 빠른 TTFB (첫 바이트를 받는 것 까지의 시간)를 가지고 있으며 항상 새로운 데이터가 서빙된다. 우리는 정적 빌드(static build) 대신에 엣지에 있는 분산 시스템을 이용하였다. 웹사이트에 오타가 있을 때 고치면 몇 초 안에 사이트에 반영이 된다. 다시 빌드할 필요도, 다시 배포할 필요도, 심지어 HTTP Caching 할 필요도 없다.

빠르게 만들 수 없는건 유저의 네트워크다. 유일하게 이 문제에 대한 해결 방향은 네트워크를 통해 보내는 파일의 수를 줄이는 수 밖에는 없다. 더 적은 JavaScript, 더 적은 JSON, 더 적은 CSS를 이루기 위해선 로직을 이전할 수 있는 서버와 점진적 향상(progressive enhancement)를 추구하는 프레임워크를 필요로 한다.

Remix에는 네트워크를 통해 더 적은 양의 파일을 보낼 수 있는 많은 방법들이 존재한다. 여기서는 레코드 리스트를 가져오는 사례 하나만 살펴보자.

[Github Gist API](https://api.github.com/gists)를 생각해보자. 이 데이터는 unpack 상태에서 75kb이며, 네트워크를 통해 압축되면 12kb가 된다. Browser에서 fetch가 이뤄지면 유저로 하여금 모든 내용을 다운 받게 만든다. 아래와 같은 코드가 사용될 것이다.

```javascript
export default function Gists() {
  let gists = useSomeFetchWrapper("https://api.github.com/gists");
  if (!gists) {
    return <Skeleton />;
  }
  return (
    <ul>
      {gists.map((gist) => (
        <li>
          <a href={gist.html_url}>
            {gist.description}, {gist.owner.login}
          </a>
          <ul>
            {Object.keys(gist.files).map((key) => (
              <li>{key}</li>
            ))}
          </ul>
        </li>
      ))}
    </ul>
  );
}
```

Remix를 이용한다면, 서버상에서 데이터를 필터링 한 다음에 유저에게 전달해줄 수 있다.

```javascript
export async function loader() {
  let res = await fetch("https://api.github.com/gists");
  let json = await res.json();
  return json.map((gist) => {
    return {
      url: gist.html_url,
      files: Object.keys(gist.files),
      owner: gist.owner.login,
    };
  });
}

export default function Gists() {
  let gists = useLoaderData();
  return (
    <ul>
      {gists.map((gist) => (
        <li>
          <a href={gist.url}>
            {gist.description}, {gist.owner}
          </a>
          <ul>
            {gist.files.map((key) => (
              <li>{key}</li>
            ))}
          </ul>
        </li>
      ))}
    </ul>
  );
}
```

이 방법은 기존의 12kB 압축, 75kB 비압축의 결과물에서 1.8kB 압축, 3.8kB 전체로 낮춘다. 이렇게 하면 20배나 데이터의 양이 줄어든 것을 볼 수 있다. 또한 Remix는 페이지가 렌더되기 전에 해당 데이터를 가져올 수 있기 때문에 우리는 모든 케이스에 스켈레톤 UI를 포함할 필요가 없어진다. 이것은 Server/Client 모델을 차용함으로 유저 네트워크로 더 적은 양의 데이터를 보냄에 따라 어떻게 애플리케이션을 더 빠르게 만들 수 있는지를 보여주는 하나의 사례에 불과하다.

## 웹 표준, HTTP, 그리고 HTML (Web Standards, HTTP, and HTML)

이 기술들은 오랜 기간 동안 있어왔다. 정말 단단한 기반을 가지고 있다. Remix는 이들을 완벽하게 이용한다. Remix는 URL을 이용한 asset 서빙, 동적 서버 렌더링, 그리고 `<link rel=prefetch>`와 같은 HTML 기능들을 HTTP Caching과 조합하여 애플리케이션을 빠르게 만들기 위한 모든 도구들을 갖춘다. Browser와 HTML은 우리가 사용하기 시작했을 때부터 지난 20년간 정말 좋아졌다.

우리는 Remix API를 최소환으로 남기고 웹 표준과 최대한 일할 수 있도록 했다. 예를들어, 우리들 자체적으로 `req/res` API를 만드는 대신, 혹은 심지어 Node API를 사용할 필요도 없이 Remix는 [Web Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 객체를 사용한다. 이 뜻은, Remix를 잘하게 되면 다른 웹 표준인 `Request`, `Response`, `URLSearchParams`, 그리고 `URL`를 잘하게 되는 것임을 뜻한다. 이제 어디에 배포를 하게 되더라도 서버상에 있기 때문에 모든 항목들은 Browser에서 사용할 수 있다.

Data mutation시 우리는 HTML form을 사용하였다. 다음 페이지를 위해 data와 asset들을 먼저 가져오기 위해 (prefetch) 우리는 `<link rel="prefetch">`를 사용하였고, 리소스를 캐싱하는 복잡한 캐싱 로직은 브라우저로 하여금 담당할 수 있도록 하였다. Remix는 만약 특정 유즈 케이스에서 브라우저 API가 제공된다면 그걸 사용한다.

## 점진적 향상 (Progressive Enhancement)

대부분의 최신 프레임워크들이 데이터에 대해 read API만 제공하는 반면, Remix는 read와 write 모두를 제공한다. HTML `<form>`은 90년대부터 Data mutation에 있어 핵심이었고, Remix는 해당 API를 사용하고 강화시킨다. 이를 통해 Remix 애플리케이션의 데이터 레이어로 하여금 페이지의 JavaScript 유무와 상관없이 작동하게 한다.

JavaScript를 추가함으로써 페이지 전환시 Remix는 두 방향에서 유저 경험을 증대시킨다:

1. JavaScript와 CSS asset을 다운로드 하거나 평가하지 않는다.
2. 레이아웃에서 변경되는 부분에 대해서만 데이터를 가져온다.

또한, JavaScript를 통해 Remix는 개발자로 하여금 페이지 전환시 더 좋은 UX를 만들 수 있는 API를 제공한다:

1. 브라우저의 돌아가는 favicon보다 좋은 Pending 상태 표시 UI
2. CRUD 데이터 액션시 Optimistic UI 추가

마지막으로, Data mutation이 Remix에 내장되어 있기에 mutation이 발생한 후에 변경되었을 수 있는 데이터를 언제 다시 가져올지 알고 있어 페이지의 다른 부분들이 만료된 정보를 보여주지 않도록 해준다.

여기의 포인트는 JavaScript 없이 애플리케이션이 돌아가는 것이 아니라 간단한 Server/Client 모델을 유지하는 것이다.

## 지나치게 추상화하지 말라 (Don't Over Abstract)

이건 우리가 좀 더 강조하고 싶은건데, Remix 이전에 우리는 지난 5년간 교육자였다. 우리의 슬로건은 *더 나은 웹사이트를 만들자*이다. 조금 더 추가하자면: *더 나은 웹사이트를 만들자, 가끔은 Remix를 이용해서*가 될 것이다. Remix를 잘하게 되면, 의도치않게 웹 개발도 잘하게 될 것이다.

Remix가 제공하는 API들은 기반에 깔려있는 Browser/HTTP/JavaScript를 쉽게 이용할 수 있도록 해주지만, 이 기술은 숨어있지 않다.

예를들어, 애플리케이션 내의 특정 레이아웃에 CSS는 `links`라는 라우트 모듈 메소드를 통해 HTML `<link>` 태그의 값을 가지고 있는 객체들의 배열을 반환하여 적용된다. 우리는 어느정도의 추상화를 통해 기반에 깔려있는 기술을 숨기지 않고 개발되는 애플리케이션의 성능을 최적화시켰다 (객체들의 모음으로 제공되기 때문에 중복 방지를 적용할 수도 있고 preloading 할 수도 있다). Remix의 `links`를 통해 asset을 prefetch 하는 방법을 배우면 어느 프레임워크를 사용하는 웹사이트에서든 prefetch 하는 방법을 알게 될 것이다.

Remix 실력을 쌓으면 동시에 Web 실력도 쌓을 수 있을 것이다.
