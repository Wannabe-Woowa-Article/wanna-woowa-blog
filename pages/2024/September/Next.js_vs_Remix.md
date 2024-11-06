## 🔗 [**Next.js vs. Remix - A Developer's Dilemma**](https://blog.saeloun.com/2024/02/21/nextjs-vs-remix/)

### 🗓️ 번역 날짜: 2024.09.23

### 🧚 번역한 크루: 러기(박정우)

---

# Next.js vs. Remix - 개발자의 고민

React 생태계는 웹 개발을 혁신하겠다는 약속을 내건 다양한 프레임워크들로 가득한 활기찬 환경입니다. 오늘은 그 중에서도 두 가지 인기 있는 프레임워크인 Next.js와 Remix를 살펴보려 합니다.

Next.js는 서버 사이드 렌더링을 위한 가장 인기 있는 React 프레임워크 중 하나로, 오랜 기간 사용되었으며 개발자가 필요로 하는 모든 기능을 제공하는 뛰어난 개발자 경험을 제공합니다.

Remix는 React Router의 창립자들이 만든 비교적 새로운 프레임워크로, 풀스택 개발 방식을 지향하며 여러 혁신적인 기능을 갖추고 있습니다. Remix가 2022년에 오픈소스로 공개된 이후, 많은 개발자들이 자신의 애플리케이션에 더 나은 프레임워크가 무엇인지 고민하게 되었습니다.

두 프레임워크 모두 인상적인 기능과 열정적인 커뮤니티를 자랑하지만, 다음 프로젝트에 선택할 승자는 과연 누구일까요?

이제 그들의 강점과 약점을 하나씩 살펴보며 우리의 선택을 도와줄 챔피언을 가려보겠습니다.

## **1. 라우팅 (Routing)**

### **Next.js**

Next.js는 두 가지 라우터를 제공합니다: App Router와 Pages Router. App Router는 React의 최신 기능인 서버 컴포넌트(Server Components)와 스트리밍(Streaming)을 사용할 수 있는 새로운 라우터입니다. Pages Router는 Next.js의 원래 라우터로, 서버 렌더링된 React 애플리케이션을 구축할 수 있게 했으며, 여전히 이전 Next.js 애플리케이션을 위해 지원됩니다.

App Router의 경우, Next.js 13은 디렉터리 기반 라우팅을 사용합니다. /app 아래에 있는 page.tsx라는 파일은 라우트로 빌드됩니다. app 디렉터리 내 폴더에는 레이아웃을 위한 layout.tsx, 해당 라우트를 공개적으로 접근 가능하게 만드는 page.tsx, 로딩 상태를 정의하는 loading.tsx, 그리고 오류 처리를 위한 error.tsx 파일이 포함될 수 있습니다. 중첩된 라우트를 만들기 위해서는 폴더를 서로 중첩하여 구성할 수 있습니다.

![Source: [Next.js docs](https://nextjs.org/docs/app/building-your-application/routing#route-segments)](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Froute-segments-to-path-segments.png&w=3840&q=75&dpl=dpl_BHPojYYHreNfQggWAzwexKkKktRt)

![Source: [Next.js docs](https://nextjs.org/docs/app/building-your-application/routing#component-hierarchy)](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fnested-file-conventions-component-hierarchy.png&w=3840&q=75&dpl=dpl_BHPojYYHreNfQggWAzwexKkKktRt)

### **Remix**

Remix v2는 평면 파일 기반의 라우팅 시스템을 사용합니다. /app/routes 폴더에 새로운 컴포넌트를 추가함으로써 새로운 라우트를 생성할 수 있습니다. 중첩된 라우트를 생성하려면 파일 이름에 마침표(.) 구분자를 사용합니다. 예를 들어, Remix 애플리케이션에서 /concerts/trending 라우트를 만들고 싶다면, concerts.trending.tsx라는 새 파일을 추가하면 됩니다.

```
app/
├── routes/
│   ├── _index.tsx
│   ├── about.tsx
│   ├── concerts._index.tsx
│   ├── concerts.$city.tsx
│   ├── concerts.trending.tsx
│   └── concerts.tsx
└── root.tsx
```

All the routes that start with app/routes/concerts. will be child routes of app/routes/concerts.tsx.

```
URL	Matched Route	Layout
/	app/routes/_index.tsx	app/root.tsx
/about	app/routes/about.tsx	app/root.tsx
/concerts	app/routes/concerts._index.tsx	app/routes/concerts.tsx
/concerts/trending	app/routes/concerts.trending.tsx	app/routes/concerts.tsx
/concerts/salt-lake-city	app/routes/concerts.$city.tsx	app/routes/concerts.tsx
```

Source: [Remix docs](https://remix.run/docs/en/main/discussion/routes#conventional-route-configuration)

### **관점 (Point of View)**

이제 두 프레임워크의 라우팅 메커니즘을 비교해보면, 두 프레임워크 모두 파일 시스템 기반 라우팅을 선택했고, 이는 올바른 방향으로 보입니다.

Remix는 더 직관적으로 보이는데, 파일이나 레이아웃을 보는 것만으로도 해당 라우트가 무엇을 나타내는지 알 수 있습니다. 반면, Next.js의 방식도 합리적인데, 관련된 라우팅 파일을 하나의 폴더에 넣음으로써 각 라우트 세그먼트에 대한 로딩/오류 상태를 정의하는 데 도움이 됩니다.

## **2. 데이터 Fetching**

### **Next.js**

Next.js는 여러 가지 데이터 페칭 방법을 제공합니다:

- **getServerSideProps**: 각 요청 시 서버에서 데이터를 페칭합니다. 이는 서버 사이드 렌더링(SSR)에 사용되며, 클라이언트가 페이지를 요청할 때 데이터를 가져옵니다.
- **getStaticProps**: 빌드 시 데이터를 페칭하여 사전 렌더링된 콘텐츠와 함께 정적 HTML 페이지를 생성합니다.
- **getInitialProps**: 서버와 클라이언트 모두에서 실행되며, 초기 렌더링과 클라이언트 측 하이드레이션을 위한 데이터 페칭을 가능하게 합니다. 이는 레거시 API입니다.
- **fetch**: Next.js는 기본 fetch 웹 API를 확장하여, 서버에서 각 fetch 요청에 대한 캐싱 및 재검증 동작을 구성할 수 있게 합니다. async/await와 함께 사용하는 fetch는 서버 컴포넌트, 라우트 핸들러, 서버 액션에서 사용할 수 있습니다.

```tsx
async function getUsers() {
  const res = await fetch("https://jsonplaceholder.typicode.com/users");
  if (!res.ok) {
    throw new Error("Failed to fetch data");
  }
  return res.json();
}

export default async function Page() {
  const users = await getUsers();
  return (
    <div>
      <h1>Users</h1>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

### **Remix**

Remix에서는 데이터를 로더(loaders)에서 페칭합니다. 각 라우트는 로더 함수를 정의할 수 있으며, 이 함수는 렌더링할 때 해당 라우트에 필요한 데이터를 제공합니다. **useLoaderData**는 로더의 데이터를 컴포넌트에 전달해줍니다. 로더는 서버에서만 실행됩니다.

```tsx
import { useLoaderData } from "@remix-run/react";

export const loader = async () => {
  const users = await getUsers();
  return json({ users });
};

export default function Page() {
  const users = useLoaderData<typeof loader>();
  return (
    <div>
      <h1>Users</h1>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

### **관점 (Point of View)**

Next.js는 정적 콘텐츠와 동적 콘텐츠가 혼합된 애플리케이션에 이상적이며, 유연성과 맞춤화가 중요한 경우에 적합해 보입니다. Remix의 데이터 페칭 방식은 데이터 로딩과 의존성에 대한 더 세밀한 제어를 가능하게 합니다.

## **3. 데이터 변경 (Data mutations)**

데이터 변경을 다룰 때는 종종 백엔드 서버에 API 요청을 보내고, 이후 로컬 상태를 업데이트하여 변경 사항을 반영합니다.

두 프레임워크 모두 데이터 변경 처리 방식을 혁신하여 이를 프레임워크의 핵심 기능에 직접 통합하려고 합니다.

### **Next.js**

Next.js 13.4 이전에는, 서버에서 작업을 수행하고 상태를 업데이트하는 유일한 방법은 API 라우트를 생성하는 것이었습니다.

Next.js 13.4에서는 데이터 변경을 처리하는 **서버 액션**(server actions)이 도입되어 개발자 경험을 단순화하고 사용자 경험을 향상시켰습니다.

```tsx
// Using API route
export default function Page() {
  async function onSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);
    const response = await fetch("/api/submit", {
      method: "POST",
      body: formData,
    });

    // Handle response if necessary
    const data = await response.json();
    // ...
  }

  return (
    <form onSubmit={onSubmit}>
      <input type="text" name="name" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

```tsx
//Using server actions
export default function Page() {
  async function create(formData: FormData) {
    "use server";
    const id = await createItem(formData);
  }

  return (
    <form action={create}>
      <input type="text" name="name" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### **Remix**

Remix는 자동으로 UI를 지속적인 서버 상태와 동기화합니다. 이는 세 가지 단계로 이루어집니다:

1. 라우트 로더가 UI에 데이터를 제공합니다.
2. 폼이 데이터를 라우트 액션에 전송하여 지속적인 상태를 업데이트합니다.
3. 페이지의 로더 데이터가 자동으로 재검증됩니다.

![Remix](https://remix.run/blog-images/posts/remix-data-flow/loader-action-component.png)

Remix는 사용자가 작업을 수행하는 모든 부분을 HTML 폼으로 유지하도록 권장합니다. 사용자가 폼 제출을 트리거할 때마다, 액션이 호출됩니다. 액션이 실행된 후, Remix는 해당 라우트의 모든 로더를 브라우저의 fetch 요청을 통해 다시 가져오고, UI를 새로 고쳐 UI가 항상 데이터베이스와 동기화되도록 보장합니다. 이를 Remix의 "풀스택 데이터 흐름"이라고 부릅니다.

```tsx
export async function loader({ request }: LoaderFunctionArgs) {
  const user = await getUser(request);
  return json({
    displayName: user.displayName,
    email: user.email,
  });
}

export default function Component() {
  const user = useLoaderData<typeof loader>();
  return (
    <Form method="post" action="/account">
      <h1>Settings for {user.displayName}</h1>

      <input name="displayName" defaultValue={user.displayName} />
      <input name="email" defaultValue={user.email} />

      <button type="submit">Save</button>
    </Form>
  );
}

export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData();
  const user = await getUser(request);

  await updateUser(user.id, {
    email: formData.get("email"),
    displayName: formData.get("displayName"),
  });

  return json({ ok: true });
}
```

### **관점 (Point of View)**

Next.js의 서버 액션은 React 생태계 및 React API와 밀접하게 연관되어 있습니다. 반면 Remix는 웹 플랫폼의 동작에 기반을 두며, 웹의 작동 방식과 긴밀하게 연관되어 있습니다.

Next.js의 액션은 **컴포넌트 중심**인 반면, Remix의 액션은 **라우트 중심**으로 동작하기 때문에 컴포넌트처럼 재사용성이 높은 구조는 아닙니다.

또한, Next.js에서는 경로를 다시 검증하려면 수동으로 지정해야 하지만, Remix는 자동으로 재검증을 수행합니다.

이러한 차이점들은 Next.js와 Remix 간의 트레이드오프입니다. 우리는 어떤 트레이드오프를 수용할 수 있고, 어떤 기능이 필요한지를 기준으로 선택할 수 있습니다.

## **4. 에러 처리 (Error handling)**

Next.js와 Remix는 모두 웹 애플리케이션에서 에러를 우아하게 처리할 수 있는 메커니즘을 제공합니다.

### **Next.js**

각 라우트 세그먼트에는 에러 상태를 렌더링하기 위한 별도의 `error.js` 파일이 있습니다. 이 파일은 경로 세그먼트와 중첩된 자식들을 자동으로 React Error Boundary로 감싸, 예상치 못한 런타임 에러를 우아하게 처리할 수 있게 합니다. 이 파일은 서버나 브라우저에서 발생할 수 있는 예상치 못한 에러와 404 같은 예측 가능한 에러를 모두 처리합니다.

### **Remix**

라우트 세그먼트의 에러 상태를 렌더링하기 위해 `ErrorBoundary`를 내보낼 수 있습니다. 이것은 서버나 브라우저에서 발생할 수 있는 예상치 못한 에러와 404 같은 예측 가능한 에러를 모두 처리합니다.

## **5. 커뮤니티 지원 (Community Support)**

### **Next.js**

Next.js는 118k⭐의 GitHub 별(작성 시점 기준 125k⭐)을 보유한 잘 확립된 프레임워크입니다. 커뮤니티와 생태계가 크며, 문제 해결, 플러그인, 통합 등을 찾을 때 큰 이점이 됩니다.

### **Remix**

Remix는 약 26.6k⭐의 GitHub 별(작성 시점 기준 29.4k⭐)을 보유하고 있으며, 커뮤니티가 성장 중입니다.

### **관점 (Point of View)**

애플리케이션이 복잡하지 않고 커뮤니티의 도움을 많이 필요로 하지 않는다면 Remix를 선호할 수 있습니다. 반면, 더 넓은 기능 범위와 큰 커뮤니티가 필요한 애플리케이션에는 Next.js가 좋은 선택입니다.

---

## **6. 학습 곡선 (Learning Curve)**

### **Next.js**

상대적으로 배우기 어렵습니다. 다양한 선택지와 저수준 제어 기능을 제공하지만, 이를 제대로 사용하지 않으면 과도할 수 있습니다.

## **Remix**

상대적으로 간단합니다. 단일 방식으로 작업을 처리하며, 많은 것을 추상화하여 제공됩니다

## **7. 배포 (Deployment)**

### **Next.js**

Vercel 외의 플랫폼에서 Next.js를 배포하는 것은 도전적일 수 있습니다. Vercel은 훌륭한 플랫폼이지만, 인프라가 AWS에 있다면 이상적이지 않을 수 있습니다. AWS 계정에서 Next.js를 호스팅하면 백엔드와의 통합이 더 쉬워지고, Vercel보다 비용 효율적인 경우가 많습니다. Next.js는 서버리스 환경에서 자체 호스팅을 위한 기본 지원이 없지만, Node 애플리케이션으로 실행할 수 있습니다. 하지만 이 방법은 Vercel을 사용하는 것과 동일한 이점을 제공하지 않을 수 있습니다.

다행히 새로운 오픈 소스 Next.js 서버리스 어댑터인 **OpenNext**가 있습니다. 이 어댑터는 Next.js 빌드 출력을 가져와 FaaS(Functions as a Service) 플랫폼에 배포할 수 있는 패키지로 변환하여 배포를 더 유연하게 만듭니다.

Kent Dodds는 자신의 블로그에서 Next.js 배포에 대한 우려를 표현하기도 했습니다.

### **Remix**

Remix는 자바스크립트 실행을 지원하는 모든 플랫폼에 배포할 수 있도록 설계되었습니다. 이는 표준에 중점을 둔 Remix의 특성 덕분입니다.

## **8. 가격 (Pricing)**

### **Next.js**

Vercel의 가격 정책은 많은 사람들에게 문제로 지적되고 있습니다. 이는 고려해야 할 중요한 요소가 될 수 있습니다.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">We we&#39;re quoted $40k for ▲ (after rejecting 150k for enterprise tier) for just static asset applications we wanted to move.<br><br>Our current Cloudfront bill per month: 12$</p>&mdash; Zackery Griesinger (@zackerydev) <a href="https://twitter.com/zackerydev/status/1717556827569660378?ref_src=twsrc%5Etfw">October 26, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

**하지만** Vercel의 제품 부사장인 Lee Rob는 게시물에서 가격 정책을 개선하기 위해 노력하고 있다고 언급했습니다.

### **Remix**

Remix는 자바스크립트 실행을 지원하는 모든 플랫폼에 배포할 수 있으므로, 원하는 플랫폼을 자유롭게 선택할 수 있습니다.

---

## **9. 대기업과의 파트너십 (Partnership with big brands)**

### **Next.js**

Next.js는 Vercel이 유지 관리하며, React 팀이 Next.js 팀과 긴밀히 협력하여 React Server Components와 같은 새로운 기능을 출시하고 있습니다.

### **Remix**

Remix는 2022년에 Shopify와 파트너십을 맺었습니다! Shopify의 후원 하에, Remix는 상거래 분야의 선도 기업으로부터 장기적인 지원을 받게 되었습니다.

## **10. 사용 기업 (Companies)**

### **Next.js**

- 넷플릭스(Netflix Jobs)
- 틱톡(TikTok)
- 노션(Notion)
- 룸(Loom)자세한 목록은 [여기](https://www.notion.so/Next-js-vs-Remix-A-Developer-s-Dilemma-10a785ff8231801c8e60d06ec171a753?pvs=21)에서 볼 수 있습니다.

### **Remix**

- NASA
- 도커(Docker) - Docker Scout는 모든 저장소에서 취약점을 빠르게 식별하고 수정할 수 있는 통합 컨테이너 보안 솔루션입니다.
- Shopify
- React-admin - 비공개 npm 레지스트리와 기업용 대시보드를 제공합니다.

자세한 목록은 [여기](https://remix.run/showcase)에서 볼 수 있습니다.

**그래서 승자는 누구일까요?**

그리고 우승자는...

**무승부**입니다! Next.js와 Remix 모두 각기 다른 분야에서 뛰어난 성과를 보이고 있습니다.

그러나 "최고"의 프레임워크는 프로젝트의 고유한 요구 사항에 따라 달라집니다:

- **Next.js**: 대규모 프로젝트, 기능이 풍부한 프레임워크, 광범위한 지원으로 빠르게 성과를 내야 하는 경우에는 Next.js가 적합할 수 있습니다.
- **Remix**: 성능이 중요한 프로젝트, 부드러운 사용자 경험, 복잡하지 않은 문제 해결, 현대적인 접근 방식을 탐구할 의향이 있다면 Remix가 더 적합할 수 있습니다.

---

**기억하세요:**

두 프레임워크 모두 활발한 커뮤니티와 성장하는 리소스를 보유하고 있습니다. 직접 실험하는 것이 중요합니다. 각각의 프레임워크로 작은 프로젝트를 만들어보면서 어떤 것이 더 잘 맞는지 확인하세요. 팀의 기술과 선호도도 중요하니, 팀의 개발 스타일과 일치하는 프레임워크를 선택하는 것이 좋습니다.

행복한 코딩 되세요!
