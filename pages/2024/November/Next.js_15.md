## 🔗 [\*\*Next.js 15](https://nextjs.org/blog/next-15)

### 🗓️ 번역 날짜: 2024.11.07

### 🧚 번역한 크루: 러기(박정우)

---

# Next.js 15

Next.js 15가 공식적으로 안정화되어 프로덕션 환경에서 사용할 준비가 되었습니다. 이번 릴리스는 [RC1](https://nextjs.org/blog/next-15-rc)과 [RC2](https://nextjs.org/blog/next-15-rc2)의 업데이트를 기반으로 하며, 안정성에 중점을 두면서도 여러분이 좋아하실 만한 흥미로운 기능들을 추가했습니다. 오늘 바로 Next.js 15를 시도해 보세요.

```
# Use the new automated upgrade CLI
npx @next/codemod@canary upgrade latest

# ...or upgrade manually
npm install next@latest react@rc react-dom@rc
```

다가오는 목요일, 10월 24일에 열리는 Next.js Conf에서 곧 다가올 업데이트에 대해 더 많은 내용을 공유할 예정입니다.

Next.js 15의 새로운 기능은 다음과 같습니다.

- @next/codemod CLI: 최신 Next.js 및 React 버전으로 손쉽게 업그레이드할 수 있는 CLI 도구.
- 비동기 요청 API (Breaking): 렌더링 및 캐싱 모델을 단순화하기 위한 단계적 접근.
- 캐싱 방식 (Breaking): fetch 요청, GET 라우트 핸들러, 클라이언트 탐색이 기본적으로 캐시되지 않음.
- React 19 지원: React 19, React Compiler (실험적), 및 hydration 오류 개선 지원.
- Turbopack Dev (Stable): 성능 및 안정성 향상.
- 정적 표시기: 개발 중 정적 라우트를 시각적으로 표시하는 새 인디케이터.
- unstable_after API (Experimental): 응답이 스트리밍 완료된 후 코드를 실행하는 실험적 API.
- instrumentation.js API (Stable): 서버 수명 주기 가시성을 위한 새로운 API.
- 향상된 폼 (next/form): HTML 폼에 클라이언트 측 탐색 기능 추가.
- next.config: next.config.ts에서 TypeScript 지원.
- 셀프 호스팅 개선: Cache-Control 헤더에 대한 더 많은 제어 옵션 제공.
- 서버 액션 보안: 추측 불가능한 엔드포인트와 사용하지 않는 액션 제거.
- 외부 패키지 번들링 (Stable): 앱 및 페이지 라우터용 새 구성 옵션.
- ESLint 9 지원: ESLint 9 지원 추가.
- 개발 및 빌드 성능: 더 빠른 빌드 시간과 향상된 Fast Refresh.

## 원활한 업그레이드를 위한 @next/codemod CLI

모든 주요 Next.js 릴리스에는 codemod라는 자동 코드 변환 도구가 포함되어 있어, 주요 변경 사항을 손쉽게 업그레이드할 수 있도록 지원합니다.

이번에는 업그레이드 과정을 더욱 매끄럽게 돕기 위해, 향상된 codemod CLI를 출시했습니다.

```
npx @next/codemod@canary upgrade latest
```

이 도구는 코드베이스를 최신 안정 버전 또는 사전 릴리스 버전으로 업그레이드하는 데 도움을 줍니다. CLI는 종속성을 업데이트하고, 사용 가능한 codemod를 표시하며, 이를 적용하는 과정을 안내합니다.

canary 태그는 codemod의 최신 버전을 사용하며, latest는 Next.js 버전을 지정합니다. 최신 Next.js 버전으로 업그레이드하더라도, 사용자 피드백을 기반으로 도구를 지속적으로 개선할 계획이므로 canary 버전의 codemod를 사용하는 것을 권장합니다.

Next.js codemod CLI에 대한 자세한 내용은 [Next.js 공식 문서](https://nextjs.org/docs/app/building-your-application/upgrading/codemods)를 참조하세요.

## 비동기 Request API (주요 변경 사항)

기존 서버사이드 렌더링(SSR)에서는 서버가 요청을 기다렸다가 콘텐츠를 렌더링합니다. 하지만 모든 컴포넌트가 요청에 따른 특정 데이터에 의존하지 않으므로, 요청을 기다리지 않고 렌더링할 수 있는 경우 기다릴 필요가 없습니다. 이상적으로는 서버가 요청이 도착하기 전에 가능한 한 많은 준비를 마쳐야 합니다. 이를 가능하게 하고 미래의 최적화를 위해, 언제 요청을 기다려야 할지를 서버가 알 필요가 있습니다.

따라서, 헤더, 쿠키, params, searchParams와 같이 요청에 따라 달라지는 데이터에 의존하는 API를 비동기로 전환하고 있습니다.

```tsx
import { cookies } from "next/headers";

export async function AdminPanel() {
  const cookieStore = await cookies();
  const token = cookieStore.get("token");

  // ...
}
```

이번 변경은 주요 변경 사항으로, 다음 API에 영향을 미칩니다.

- cookies
- headers
- draftMode
- params (layout.js, page.js, route.js, default.js, generateMetadata, generateViewport 파일들에서)
- searchParams (page.js에서)

더 쉽게 마이그레이션할 수 있도록, 이러한 API는 일시적으로 동기 방식으로 접근이 가능하지만, 다음 주요 버전까지 개발 및 프로덕션 환경에서 경고 메시지가 표시됩니다. 마이그레이션을 자동화하기 위한 [codemod](https://nextjs.org/docs/app/building-your-application/upgrading/codemods)도 제공됩니다.

```
npx @next/codemod@canary next-async-request-api .
```

codemod가 코드의 완전한 마이그레이션을 수행하지 못하는 경우, [업그레이드 가이드](https://nextjs.org/docs/app/building-your-application/upgrading/version-15)를 참고해 주세요. 새로운 API로 Next.js 애플리케이션을 마이그레이션하는 [예제](https://github.com/leerob/next-saas-starter/pull/62)도 함께 제공하고 있습니다.

## 캐싱 방식

Next.js App Router는 기본적으로 성능을 최적화한 캐싱 방식을 제공하여 필요 시 캐싱을 선택적으로 해제할 수 있도록 설계되었습니다.

사용자 피드백을 바탕으로, Partial Prerendering(PPR)과 fetch를 사용하는 서드파티 라이브러리와의 상호 작용 방식을 고려하여 [캐싱 규칙](https://x.com/feedthejim/status/1785242054773145636)을 재검토했습니다.

Next.js 15에서는 GET 라우트 핸들러와 클라이언트 라우터 캐시의 기본 설정을 캐싱된 상태에서 비캐싱 상태로 변경했습니다. 이전의 캐싱 동작을 유지하려면, 직접 캐싱을 선택할 수 있습니다.

앞으로 몇 달 간 Next.js의 캐싱 기능을 지속적으로 개선할 예정이며, 더 많은 정보를 곧 공유하겠습니다.

### GET 라우트 핸들러는 기본적으로 캐싱되지 않음

Next.js 14에서는 GET HTTP 메서드를 사용하는 라우트 핸들러가 동적 함수나 동적 구성 옵션을 사용하지 않는 한 기본적으로 캐싱되었습니다. 하지만 Next.js 15부터는 GET 함수가 기본적으로 캐싱되지 않습니다.

여전히 `export const dynamic = 'force-static'`과 같은 정적 라우트 구성 옵션을 사용하여 캐싱을 활성화할 수 있습니다.

sitemap.ts, opengraph-image.tsx, icon.tsx와 같은 특별한 라우트 핸들러와 기타 메타데이터 파일은 동적 함수나 동적 구성 옵션을 사용하지 않는 한 기본적으로 정적으로 유지됩니다.

### 클라이언트 라우터 캐시가 기본적으로 페이지 컴포넌트를 캐싱하지 않음

Next.js 14.2.0에서는 라우터 캐시의 사용자 지정 구성을 위해 실험적인 staleTimes 플래그를 도입했습니다.

Next.js 15에서는 이 플래그가 여전히 사용 가능하지만, 페이지 세그먼트의 기본 staleTime 값을 0으로 변경했습니다. 즉, 앱을 탐색할 때마다 활성화된 페이지 컴포넌트의 최신 데이터가 항상 반영됩니다. 그러나 다음과 같은 중요한 동작은 변경되지 않습니다.

- 공유 레이아웃 데이터: 부분 렌더링을 지원하기 위해 서버에서 다시 가져오지 않습니다.
- 뒤로/앞으로 탐색: 스크롤 위치 복원을 위해 캐시에서 복원됩니다.
- loading.js: 5분간(또는 staleTimes.static 설정 값) 캐싱된 상태를 유지합니다.

이전 클라이언트 라우터 캐시 동작을 사용하려면 다음 설정을 통해 선택할 수 있습니다.

```ts
const nextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30,
    },
  },
};

export default nextConfig;
```

## React 19

Next.js 15 릴리스의 일환으로, 곧 출시될 React 19와 호환되도록 업데이트를 진행했습니다.

버전 15에서 App Router는 React 19 RC를 사용하며, 커뮤니티 피드백을 반영해 Pages Router에 React 18에 대한 하위 호환성도 추가했습니다. Pages Router를 사용하는 경우, 준비가 되면 React 19로 업그레이드할 수 있습니다.

React 19는 아직 RC 단계에 있지만, 실제 애플리케이션에서의 광범위한 테스트와 React 팀과의 긴밀한 협력을 통해 안정성을 확신하게 되었습니다. 주요 변경 사항은 충분히 검증되어 기존 App Router 사용자에게 영향을 주지 않습니다. 이에 따라 Next.js 15를 안정 버전으로 출시하여, 프로젝트가 React 19 GA에 대비할 수 있도록 준비했습니다.

원활한 전환을 위해, 마이그레이션을 돕는 codemod와 자동화 도구도 제공하고 있습니다.

자세한 내용은 [Next.js 15 업그레이드 가이드](https://nextjs.org/docs/app/building-your-application/upgrading/version-15), [React 19 업그레이드 가이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide), 그리고 [React Conf Keynote](https://www.youtube.com/live/T8TZQ6k4SLE?t=1788s)를 통해 확인해 보세요.

## Pages Router와 React 18

Next.js 15는 Pages Router에 대해 React 18의 하위 호환성을 유지하여, 사용자가 Next.js 15의 개선 사항을 활용하면서도 React 18을 계속 사용할 수 있도록 합니다.

RC1 이후 커뮤니티 피드백을 반영하여 React 18 지원을 포함하도록 업데이트했으며, 이를 통해 Pages Router와 함께 React 18을 사용하면서 Next.js 15로 업그레이드할 수 있는 유연성을 제공합니다. 이로써 업그레이드 경로에 대한 더 큰 선택권을 갖게 됩니다.

> 참고: 하나의 애플리케이션에서 Pages Router를 React 18로, App Router를 React 19로 사용하는 것이 가능하지만, 이러한 설정은 권장되지 않습니다. 이러한 혼합 사용은 예기치 않은 동작이나 타입 불일치가 발생할 수 있으며, 두 버전 간의 API 및 렌더링 로직이 완전히 일치하지 않을 수 있기 때문입니다.

React Compiler는 Meta의 React 팀이 개발한 새로운 실험적 컴파일러로, JavaScript의 의미와 React의 규칙을 깊이 이해하여 코드를 자동으로 최적화합니다. 이를 통해 개발자가 useMemo나 useCallback과 같은 API를 사용하여 수동으로 메모이제이션해야 하는 작업을 줄여, 코드를 더 간단하고 유지보수하기 쉽게 만듭니다.

Next.js 15에서는 React Compiler에 대한 지원이 추가되었습니다. 자세한 내용과 Next.js의 구성 옵션은 Next.js 공식 문서를 참고하세요.

주의: 현재 React Compiler는 Babel 플러그인으로만 사용 가능하며, 이로 인해 개발 및 빌드 시간이 느려질 수 있습니다.

## React Compiler (실험적)

[React Compiler](https://react.dev/learn/react-compiler)는 Meta의 React 팀이 개발한 새로운 실험적 컴파일러로, JavaScript의 의미와 [React 규칙](https://react.dev/reference/rules)을 깊이 이해하여 코드를 자동으로 최적화할 수 있습니다. 이를 통해 개발자는 useMemo와 useCallback 같은 API를 이용해 수동으로 메모이제이션을 설정해야 하는 작업을 줄일 수 있으며, 코드가 더 간단하고 유지 관리하기 쉬워져 오류 발생 가능성도 낮아집니다.

Next.js 15에서는 React Compiler에 대한 지원이 추가되었습니다. React Compiler와 [Next.js에서 제공하는 구성 옵션에 대해 자세히 알아보세요](https://react.dev/learn/react-compiler#usage-with-nextjs).

참고: React Compiler는 현재 Babel 플러그인으로만 제공되므로, 개발 및 빌드 속도가 느려질 수 있습니다.

## 하이드레이션 오류 개선

Next.js 14.1에서는 오류 메시지와 하이드레이션 오류를 개선했으며, Next.js 15에서는 이를 기반으로 더 향상된 하이드레이션 오류 화면을 추가했습니다. 이제 하이드레이션 오류가 발생하면, 오류가 발생한 소스 코드와 함께 문제 해결을 위한 권장 사항이 표시됩니다.

예를 들어, Next.js 14.1에서의 하이드레이션 오류 메시지는 다음과 같았습니다.

![Next.js 14.1의 하이드레이션 오류 메시지](https://nextjs.org/_next/image?url=%2Fstatic%2Fblog%2Fnext-15-rc%2Fhydration-error-before-dark.png&w=2048&q=75&dpl=dpl_Bp7ZN8sXBC4Jwi5G139cNcFGQWd9)

Next.js 15에서는 이를 개선하여 다음과 같은 메시지를 제공합니다.

![Next.js 15에서 개선된 하이드레이션 오류 메시지](https://nextjs.org/_next/image?url=%2Fstatic%2Fblog%2Fnext-15-rc%2Fhydration-error-after-dark.png&w=1920&q=75&dpl=dpl_Bp7ZN8sXBC4Jwi5G139cNcFGQWd9)

## Turbopack Dev

next dev --turbo 명령어가 이제 안정화되어 개발 경험을 한층 빠르게 개선할 수 있게 되었습니다. 저희는 이 기능을 vercel.com, nextjs.org, [v0](https://v0.dev/)를 비롯한 여러 애플리케이션에서 적용하며 좋은 성능 향상을 확인했습니다.

예를 들어, 대규모 Next.js 애플리케이션인 vercel.com에서는 다음과 같은 성능 개선이 있었습니다:

- 로컬 서버 시작 속도 최대 76.7% 향상
- Fast Refresh를 통한 코드 업데이트 속도 최대 96.3% 향상
- 캐싱 없이 초기 라우트 컴파일 속도 최대 45.8% 향상 (현재 Turbopack은 디스크 캐싱을 지원하지 않음)

Turbopack Dev에 대한 자세한 내용은 새로운 블로그 게시물에서 확인할 수 있습니다. [블로그 글](https://nextjs.org/blog/turbopack-for-development-stable)

## 정적 라우트 표시기

Next.js는 이제 개발 중에 정적 라우트 표시기를 제공하여, 어떤 라우트가 정적인지 또는 동적인지 쉽게 식별할 수 있습니다. 이러한 시각적 안내를 통해 페이지가 어떻게 렌더링되는지 이해하여 성능 최적화에 도움이 됩니다.

![정적 라우트 표시기](https://nextjs.org/_next/image?url=%2Fstatic%2Fblog%2Fnext-15-rc2%2Fstatic-route-dark.png&w=3840&q=75&dpl=dpl_7FjfvzqCCJ1s444NjWcTm3VGJGVj)

빌드 결과물에서 모든 라우트의 렌더링 전략을 [next build](https://nextjs.org/docs/app/api-reference/cli/next#next-build-options)를 통해 확인할 수도 있습니다.

이번 업데이트는 Next.js의 가시성을 높이기 위한 지속적인 노력의 일환으로, 개발자가 애플리케이션을 모니터링하고 디버깅하며 최적화하기 쉽게 만들어줍니다. 전용 개발 도구도 준비 중이며, 자세한 내용은 곧 공개될 예정입니다.

[정적 라우트 표시기](https://nextjs.org/docs/app/api-reference/next-config-js/devIndicators#appisrstatus-static-indicator)에 대해 더 알아보세요. 필요시 비활성화할 수도 있습니다.

## unstable_after를 사용한 응답 후 코드 실행 (실험적)

사용자 요청을 처리할 때 서버는 일반적으로 응답을 계산하는 데 직접 관련된 작업을 수행합니다. 그러나 로그 기록, 분석, 외부 시스템 동기화와 같은 작업을 처리해야 할 수도 있습니다.

이러한 작업은 응답과 직접적으로 관련이 없기 때문에 사용자가 완료되기를 기다릴 필요가 없습니다. 응답 이후에 작업을 지연하려고 해도 서버리스 함수는 응답이 닫힌 후 즉시 처리를 중지하므로 어려움이 발생합니다.

after()는 응답이 스트리밍을 마친 후 작업을 예약할 수 있도록 해주는 새로운 실험적 API로, 보조 작업이 주요 응답을 방해하지 않고 실행될 수 있도록 합니다.

사용하려면 next.config.js에 experimental.after를 추가하세요.

```
// next.config.ts
const nextConfig = {
  experimental: {
    after: true,
  },
};

export default nextConfig;
```

그런 다음, 서버 컴포넌트, 서버 액션, 라우트 핸들러 또는 미들웨어에서 이 함수를 import합니다.

```
import { unstable_after as after } from 'next/server';
import { log } from '@/app/utils';

export default function Layout({ children }) {
  // 보조 작업
  after(() => {
    log();
  });

  // 주요 작업
  return <>{children}</>;
}
```

unstable_after에 대한 자세한 내용은 [여기](https://nextjs.org/docs/app/api-reference/functions/unstable_after)를 참조하세요.

## instrumentation.js (안정 버전)

instrumentation.js 파일과 register() API를 통해 사용자들은 Next.js 서버의 라이프사이클에 접근하여 성능을 모니터링하고, 오류의 근원을 추적하며 OpenTelemetry와 같은 가시성(Observability) 라이브러리와 깊이 통합할 수 있습니다.

이 기능은 이제 안정화되었으며, experimental.instrumentationHook 구성 옵션을 제거해도 됩니다.

또한 Sentry와 협력하여 새로운 onRequestError 훅을 설계했으며, 이를 통해 다음을 수행할 수 있습니다.

- 서버에서 발생하는 모든 오류에 대한 중요한 컨텍스트를 수집합니다. 수집 항목에는 다음이 포함됩니다.
  - 라우터: Pages Router 또는 App Router
  - 서버 컨텍스트: 서버 컴포넌트, 서버 액션, 라우트 핸들러 또는 미들웨어
- 수집한 오류 정보를 선호하는 가시성 제공자에 보고할 수 있습니다.

```
export async function onRequestError(err, request, context) {
  await fetch('https://...', {
    method: 'POST',
    body: JSON.stringify({ message: err.message, request, context }),
    headers: { 'Content-Type': 'application/json' },
  });
}

export async function register() {
  // 선호하는 가시성 제공자 SDK 설정
}
```

onRequestError 함수에 대한 자세한 내용은 [여기](https://nextjs.org/docs/app/api-reference/file-conventions/instrumentation#onrequesterror-optional)에서 확인하세요.

## `<Form>` 컴포넌트

새로운 `<Form>` 컴포넌트는 HTML `<form>` 요소를 확장하여 프리페치, 클라이언트 측 탐색, 점진적 향상을 제공합니다.

이 컴포넌트는 결과 페이지로 이동하는 검색 폼과 같이 새로운 페이지로 이동하는 폼에 유용합니다.

```
// app/page.jsx
import Form from 'next/form';

export default function Page() {
  return (
    <Form action="/search">
      <input name="query" />
      <button type="submit">Submit</button>
    </Form>
  );
}
```

`<Form>` 컴포넌트의 주요 기능.

- 프리페치: 폼이 보이면 [레이아웃](https://nextjs.org/docs/app/api-reference/file-conventions/layout)과 [로딩](https://nextjs.org/docs/app/api-reference/file-conventions/loading) UI가 미리 로드되어 빠른 탐색을 지원합니다.
- 클라이언트 측 탐색: 폼 제출 시, 공유 레이아웃과 클라이언트 상태가 유지됩니다.
- 점진적 향상: JavaScript가 로드되지 않은 경우에도 폼은 전체 페이지 탐색을 통해 정상 작동합니다.
  이전에는 이러한 기능을 구현하기 위해 많은 수작업이 필요했지만, 이제 `<Form>` 컴포넌트로 간편하게 사용할 수 있습니다.

<details>
<summary>예시</summary>
<pre><code class="language-jsx">
// Note: This is abbreviated for demonstration purposes.
// Not recommended for use in production code.

'use client'

import { useEffect } from 'react'
import { useRouter } from 'next/navigation'

export default function Form(props) {
const action = props.action
const router = useRouter()

useEffect(() => {
// if form target is a URL, prefetch it
if (typeof action === 'string') {
router.prefetch(action)
}
}, [action, router])

function onSubmit(event) {
event.preventDefault()

    // grab all of the form fields and trigger a `router.push` with the data URL encoded
    const formData = new FormData(event.currentTarget)
    const data = new URLSearchParams()

    for (const [name, value] of formData) {
      data.append(name, value as string)
    }

    router.push(`${action}?${data.toString()}`)

}

if (typeof action === 'string') {
return <form onSubmit={onSubmit} {...props} />
}

return <form {...props} />
}

</code><pre>

</details>

[`<Form>` 컴포넌트](https://nextjs.org/docs/app/api-reference/components/form)에 대해서 더 알아보세요.

## next.config.ts 지원

Next.js는 이제 TypeScript 파일 형식인 next.config.ts를 지원하며, 자동 완성과 타입 안전성을 위한 NextConfig 타입을 제공합니다.

```tsx
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* 설정 옵션 입력 */
};

export default nextConfig;
```

TypeScript 지원에 대한 자세한 내용은 Next.js [공식 문서](https://nextjs.org/docs/app/api-reference/config/typescript)에서 확인하세요.

## 자체 호스팅 개선 사항

애플리케이션을 자체 호스팅할 때, `Cache-Control` 지시에 대한 더 많은 제어가 필요할 수 있습니다.

대표적인 경우로 ISR 페이지에 대해 `stale-while-revalidate` 기간을 제어하는 상황이 있습니다. 이를 위해 두 가지 개선 사항을 구현했습니다:

`next.config`에서 [`expireTime`](https://nextjs.org/docs/app/api-reference/next-config-js/expireTime) 값을 설정할 수 있습니다. 이는 이전의 `experimental.swrDelta` 옵션을 대체합니다.
기본값이 1년으로 업데이트되어 대부분의 CDN이 stale-while-revalidate를 원하는 대로 적용할 수 있습니다.
또한, 기본 `Cache-Control` 값을 더 이상 사용자 지정 값을 덮어쓰지 않아, CDN 설정과의 호환성을 보장하면서 완전한 제어가 가능합니다.

마지막으로 이미지 최적화 기능을 개선했습니다. 이전에는 Next.js 서버에서 이미지를 최적화하기 위해 `sharp` 설치를 권장했으나, 종종 누락되는 경우가 있었습니다. Next.js 15에서는 `next start` 명령어를 사용하거나 [독립 실행 모드](https://nextjs.org/docs/app/api-reference/next-config-js/output)에서 실행할 때 `sharp`을 자동으로 사용하므로 수동 설치가 필요하지 않습니다.

자세한 내용은 Next.js 자체 호스팅에 대한 새로운 [데모와 튜토리얼 비디오](https://x.com/leeerob/status/1843796169173995544)에서 확인하세요.

## 서버 액션의 보안 강화

서버 액션(Server Actions)은 클라이언트에서 호출할 수 있는 서버 측 함수입니다. 파일 상단에 'use server' 지시어를 추가하고 비동기 함수를 export하여 정의할 수 있습니다.

서버 액션이나 유틸리티 함수가 코드에서 다른 곳에 import되지 않더라도, 이는 여전히 공개적으로 접근 가능한 HTTP 엔드포인트가 될 수 있습니다. 이 동작은 기술적으로는 올바르지만, 의도치 않게 이러한 함수가 노출될 위험이 있습니다.

보안을 강화하기 위해 다음과 같은 개선 사항을 도입했습니다.

- 불필요한 코드 제거: 사용되지 않는 서버 액션은 클라이언트 측 번들에서 ID가 노출되지 않으며, 이로 인해 번들 크기가 줄어들고 성능이 향상됩니다.
- 보안성이 강화된 액션 ID: Next.js는 추측이 불가능하고 비결정적인 ID를 생성하여, 클라이언트가 서버 액션을 참조하고 호출할 수 있도록 합니다. 이러한 ID는 빌드 간에 주기적으로 재계산되어 보안을 강화합니다.

예제 코드.

```tsx
// app/actions.js
"use server";

// 이 액션은 애플리케이션에서 사용되므로 Next.js가
// 클라이언트에서 참조 및 호출할 수 있도록 보안 ID를 생성합니다.
export async function updateUserAction(formData) {}

// 이 액션은 애플리케이션에서 사용되지 않으므로
// Next.js가 `next build` 중에 자동으로 이 코드를 제거하고
// 공개 엔드포인트를 생성하지 않습니다.
export async function deleteUserAction(formData) {}
```

서버 액션은 여전히 공개 HTTP 엔드포인트로 취급해야 합니다. 서버 액션 보안에 대한 자세한 내용은 [여기](https://nextjs.org/blog/security-nextjs-server-components-actions#write)를 참고하세요.

## 외부 패키지 번들링 최적화 (안정 버전)

외부 패키지를 번들링하면 애플리케이션의 초기 시작 성능을 향상시킬 수 있습니다. App Router에서는 외부 패키지가 기본적으로 번들링되며, 새로운 `serverExternalPackages` 설정 옵션을 사용하여 특정 패키지를 번들링에서 제외할 수 있습니다.

Pages Router에서는 외부 패키지가 기본적으로 번들링되지 않지만, 기존 `transpilePackages` 옵션을 사용하여 번들링할 패키지 목록을 지정할 수 있습니다. 이 옵션에서는 각 패키지를 개별적으로 지정해야 합니다.

App Router와 Pages Router의 설정을 통합하기 위해, Pages Router에서 외부 패키지를 자동으로 번들링하는 `bundlePagesRouterDependencies` 옵션을 추가했습니다. 이를 통해 App Router의 기본 번들링 동작과 일치하게 설정할 수 있으며, 필요 시 `serverExternalPackages`를 사용해 특정 패키지를 번들링에서 제외할 수 있습니다.

```
// next.config.ts
const nextConfig = {
  // Pages Router에서 외부 패키지를 자동으로 번들링:
  bundlePagesRouterDependencies: true,
  // App 및 Pages Router에서 특정 패키지를 번들링에서 제외:
  serverExternalPackages: ['package-name'],
};

export default nextConfig;
```

외부 패키지 최적화에 대한 자세한 내용은 Next.js [공식 문서](https://nextjs.org/docs/app/building-your-application/optimizing/package-bundling)에서 확인하세요.

Next.js 15에서는 ESLint 9에 대한 지원을 도입하여, 2024년 10월 5일에 종료된 ESLint 8의 지원을 이어받았습니다. 이로 인해 개발자는 ESLint 8과 9 중 원하는 버전을 선택하여 사용할 수 있습니다.

## ESLint 9 지원

[ESLint 9](https://eslint.org/blog/2024/04/eslint-v9.0.0-released/)로 업그레이드할 경우, [새로운 구성 형식](https://eslint.org/blog/2024/04/eslint-v9.0.0-released/#flat-config-is-now-the-default-and-has-some-changes)을 아직 채택하지 않은 프로젝트에서는 Next.js가 자동으로 `ESLINT_USE_FLAT_CONFIG=false` 설정을 적용하여 마이그레이션을 원활하게 지원합니다.
또한, `--ext` 및 `--ignore-path`와 같은 사용 중단된 옵션은 next lint 실행 시 제거됩니다. ESLint 10에서는 이러한 이전 구성 옵션이 완전히 지원되지 않을 예정이므로, 조속한 마이그레이션을 권장합니다. [마이그레이션 가이드](https://eslint.org/docs/latest/use/migrate-to-9.0.0)

이 업데이트의 일환으로, React Hooks 사용에 대한 새로운 규칙을 도입한 eslint-plugin-react-hooks를 v5.0.0으로 업그레이드했습니다. 자세한 변경 사항은 eslint-plugin-react-hooks@5.0.0의 변경 로그에서 [확인](https://github.com/facebook/react/releases/tag/eslint-plugin-react-hooks%405.0.0)하실 수 있습니다.

## 개발 및 빌드 성능 개선

서버 컴포넌트 HMR
개발 중에 서버 컴포넌트는 저장될 때마다 다시 실행됩니다. 이는 API 엔드포인트나 서드파티 서비스에 대한 fetch 요청이 반복적으로 호출될 수 있다는 뜻입니다.

로컬 개발 성능을 개선하고 API 요청에 대한 비용을 줄이기 위해, 이제 핫 모듈 교체(HMR)가 이전 렌더링에서의 fetch 응답을 재사용할 수 있도록 지원합니다.

자세한 내용은 [서버 컴포넌트 HMR 캐시](https://nextjs.org/docs/app/api-reference/next-config-js/serverComponentsHmrCache)를 참고하세요.

## App Router를 위한 더 빠른 정적 생성

정적 생성 과정을 최적화하여 특히 네트워크 요청이 느린 페이지의 빌드 시간을 개선했습니다.

이전에는 클라이언트 측 탐색 데이터를 생성하기 위해 한 번, 초기 페이지 방문을 위한 HTML 렌더링을 위해 다시 한 번 페이지를 렌더링하는 방식이었습니다. 이제 첫 번째 렌더링을 재사용하여 두 번째 렌더링을 생략함으로써 작업량과 빌드 시간을 줄였습니다.

추가로, 정적 생성 작업자들이 여러 페이지에서 fetch 캐시를 공유합니다. 캐시에서 제외되지 않은 fetch 호출의 결과는 동일한 작업자가 처리하는 다른 페이지에서도 재사용되어, 동일한 데이터 요청 횟수가 줄어듭니다.

## 고급 정적 생성 제어 (실험적)

고급 사용 사례를 위해 정적 생성 프로세스를 더 세밀하게 제어할 수 있는 실험적 지원 기능이 추가되었습니다. 하지만 특별한 요구 사항이 없다면 현재 기본값을 사용하는 것이 좋습니다. 이러한 옵션은 리소스 사용량 증가와 동시성 증가로 인해 메모리 부족 오류가 발생할 수 있기 때문입니다.

```tsx
// next.config.ts
const nextConfig = {
  experimental: {
    // 페이지 생성 실패 시, 빌드가 실패하기 전 재시도 횟수
    staticGenerationRetryCount: 1,
    // 작업자당 처리할 페이지 수
    staticGenerationMaxConcurrency: 8,
    // 새로운 작업자 스레드를 시작하기 위한 최소 페이지 수
    staticGenerationMinPagesPerWorker: 25,
  },
};

export default nextConfig;
```

정적 생성 옵션에 대한 자세한 내용은 Next.js 공식 문서를 참고하세요.
