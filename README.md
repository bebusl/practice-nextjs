# Next.js 15 실습

- [대시보드앱](https://nextjs.org/learn/dashboard-app) 튜토리얼 따라하기

## 예제 구조

    - /app : 모든 routes, 컴포넌트들, logic이 다 여기에 있음.
    - /app/lib : 유틸리티 함수랑 데이터 패칭함수같은 앱에서 범용적으로 사용하는 함수들이 여기에
    - /app/ui : UI 컴포넌트들
    - /public : 정적 에셋 포함
    - Config files: 루트에 next.config.ts같은 거 있는데, 이거 그냥 설정파일들임. 설정파일들은 루트에 있으요

## 스타일링

    - tailwind-css 쓸 수 있고, 이게 지금 제일 대중적인듯?
    - css modules : 이름에 해시값같은게 붙어서 클래스네임 충돌로 인한 스타일 오류 방지 가능.

## 폰트 최적화

    - 폰트는 웹사이트 디자인에 중요한 역할을 함. 그렇지만 페치하고 로드해야 하는 커스텀 폰트를 사용할 때 웹 페이지의 성능에 영향을 미칠 수 있음.
    - CLS => 폰트가 로딩되지 않은 경우 폴백 폰트로 노출되다가, 로드 완료후 폰트가 바꿔치기 되는데 이때 re-layout이 발생함. -> 웹 성능에 영향을 줌
    - Next.js는 next/font 모듈을 사용해서 폰트를 최적화할 수 있도록 했음. 폰트 파일을 빌드 타임에 다운로드 받고, 정적 에셋으로써 호스트함. 이 말인즉슨 유저가 너의 앱을 방문했을 때 폰트를 다운받기 위해 추가적인 네트워크 요청이 발생하지 않음을 의미함.

## 이미지 최적화

    - next.js에서 /public폴더 안에 넣으면 정적 에셋을 제공할 수 있음. 그러나 단순하게 이미지를 가져와서 사용하면 아래 작업은 매뉴얼하게 직접 해주어야 함.
      - 이미지가 서로 다른 스크린 사이즈들에 반응형으로 대응되게 해야함.
      - 다른 기기에 대응되는 이미지 사이즈들을 명시해야함
      - 이미지 로드됨에 따라 레이아웃 쉬프트가 발생하지 않도록 예방해야 함
      - 유저의 뷰포트 밖에 있는 이미지는 래이지 로드해야 함
    - 이 작업을 대신해주는게 next/image의 <Image>컴포넌트. img태그의 확장버전이며 아래와 같은 작업을 대신 해줌.
      - 이미지가 로딩 중에 레이아웃 쉬프트가 발생하지 않게 해줌
      - 작업 뷰포트에서 큰 이미지를 들고오는 것을 방지하기 위해 resizing
      - 이미지를 자동으로 레이지 로딩해줌
      - WebP, AVIF같은 모던 포맷을 제공

## 응집도 높일 수 있음

- layout.ts, pages.ts등 예약어를 제외한 애들은 라우팅 되지 않으므로, 그 페이지에서 사용하는 컴포넌트라던지 유틸함수등 가까운곳에 응집도 높게 구성할 수 있다.(By having a special name for page files, Next.js allows you to colocate UI components, test files, and other related code with your routes. Only the content inside the page file will be publicly accessible. For example, the /ui and /lib folders are colocated inside the /app folder along with your routes.
  )

## 공통 layout.ts를 적용하면 좋은점

- 상위 요소의 layout.ts를 하위 컴포넌트들도 layout.ts를 덮어씀. 다만 layout.ts는 페이지가 바뀔 때 리렌더링이 안되고, page부분만 리렌더링 됨.(레이아웃이 유지되는 동안에는, 레이아웃 내의 react state는 페이지가 바뀌어도 유지됨. - 이것과 달리 리렌더링 되는게 template.ts)

## a태그 쓰지 말고 Link로 페이지 전체 리렌더링을 막으세요!

- a태그 쓰면 페이지 전체가 리로딩되고, 리렌더링 됨. 비용이 많이 들죠? Link 컴포넌트를 이용해 client-side navigating이 되도록 해주셔요.

## 자동 코드-스플리팅과 프리페칭을 제공합니다.

- Next.js는 네비게이션 경험을 향상하기 위해 route segment를 기준으로 자동적으로 코드 스플릿을 함. -> 코드 스플릿이 된다 - 페이지별로 코드가 분리되고, 특정 페이지에서 에러가 발생하더라도 다른 부분은 여전히 정상 동작한다.
- 그리고 Link컴포넌트가 브라우저의 뷰포트에 나타나면, Next.js는 Link로 이동한 페이지에 필요한 정보를 prefetch함. 링크 클릭시 해당 페이지에 필요한 코드들은 백그라운드에서 이미 로딩되어 있기 때문에, 즉시 페이지 변환이 일어나는 것처럼 보이게 함(속도 지연 줄임)

## 데이터 페칭 방식을 고르세요

- 1. API layer를 두기
  - API는 너의 앱과 DB 중간다리 역할을 함
  - API를 제공하는 써드파티 서비스를 이용하는 경우
  - 클라이언트에 DB 시크릿키들이 노출되는 것을 방지하기 위해 서버에서 실행되는 API 레이어를 구성할 수도 있음.
  - Next.js에서는 Route Handlers를 통해 API 엔드포인트 생성 가능
- 2. DB Query로 직접 DB에 접근

  - RDBS든 NoSQL이든 다 가능
  - 만약 네가 서버 컴포넌트를 사용할 경우 API 레이어를 건너띄고 DB 쿼리를 이용해 직접 DB에 접근하는 방법 택할 수 있음. 이 경우에도 당연히 서버에서 DB에 접근하는 거기때문에 DB 시크릿키들이 노출될 염려는 하지 않아도 됨.

  - 서버 컴포넌트를 이용하는 경우
    - Next.js는 기본적으로 리액트 서버 컴포넌트를 사용함. 서버 컴포넌트에서 데이터 페칭을 하면 생기는 이점이 몇 개 있는데,
    - 1. JS Promise를 제공함. 데이터 페칭을 위해 useEffect와 useState를 쓸 필요 없음
    - 2. 서버에서 실행되므로, 비싼 데이터 페치나 로직이 서버에서 돌아가게 할 수 있음. 결과만 클라이언트로 전송
    - 3. 서버 컴포넌트는 서버에서 실행되기 때문에, API 레이어 없이 DB에 바로 접근 가능. API 레이어를 별도로 만들기 위해 작성하는 코드량과 시간을 줄일 수 있음.

## 데이터 페칭 워터폴로 인한 블로킹 해소

```
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
  numberOfInvoices,
  numberOfCustomers,
  totalPaidInvoices,
  totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

이렇게 하면 의도치 않게 워터폴이 발생함. revenue데이터 다 호출 된 뒤에, latestInvoices불러오고, 그 다음 fetchCardData를 하므로.
-> 병렬적으로 들고오면 최적화가 되겠지 그래서
`Promise.all()`또는 `Promise.allSettled()`를 써서 병렬처리를 합시다.

```
  const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);
```

- 이렇게 하면 동시에 많은 데이터 페치를 할 수 있음. -> 각 요청 완료될때까지 기다리는 시간 줄어듦
- 그러나 한 요청의 응답이 유난히 느리면 다른 애들도 다 걔를 기다려서 화면에 아무것도 안뜬다는 단점이 있음

## 정적 렌더링과 동적 렌더링

- 현재까지 논의된 내용으로 봤을 때 두가지 한계지점이 있음

  - 데이터 요청이 의도하지 않은 워터폴을 만들 수 있음
  - 대시보드의 데이터가 정적으로 생성되기 때문에, 최신 데이터가 반영되지 않을 수 있음

- 정적 렌더링(Static Rendering)

  - 빌드 타임 혹은 데이터를 revalidating할 때에 데이터 페칭이 되고, 렌더링 됨 => 앱 방문할 때마다 캐시된 결과물이 서빙됨. 그로인한 장점이 있는데
    - 1. 빠른 웹사이트 - prerender된 컨텐츠들이 캐시되고 CDN을 이용해 분산저장될 수 있음. -> 이로인해 더 빠른 접근이 가능
    - 2. 서버 로드가 줄어듦 - 컨텐츠가 다 캐시돼있기 때문에 각 유저 요청에 대해 동적으로 데이터를 생성하지 않음 => 서버 코스트가 줄겠쥬
    - 3. SEO - 미리 렌더링된 컨텐츠는 검색엔진크롤러가 인덱싱하기 좋음.(페이지 로드 시점에 이미 컨텐츠에 접근 가능하므로). 이는 검색엔진 랭킹을 올릴 수 있음

- 동적 렌더링(Dynamic Rendering)
  - 유저가 요청할 때(유저가 페이지에 방문할 때)마다 서버에서 컨텐츠를 렌더링함. 이로 인한 장점으로는
    - 실시간 데이터 - 실시간, 혹은 자주 업데이트되는 데이터를 반영할 수 있음. 데이터가 자주 변하는 앱에서는 이게 이상적
    - 유저-특정적인 컨텐츠 - 개인화된 컨텐츠를 서빙하기 쉬움.(대시보드 데이터나 유저 프로필같은 거, 유저 인터랙션에 따른 데이터 업데이트가 일어나는 경우)
    - 요청하는 시점의 정보를 알 수 있음 - request time에만 접근 가능한 정보에 접근 가능(쿠키나 URL search params같은 것들)
  - 단점 : 렌더링되기까지 가장 느린 데이터 응답 시간만큼 기다려야 함

## 스트리밍

### 스트리밍이 무엇이고 언제 사용해야할까요

- 데이터 전송 전략. -> 라우트를 작은 'chunk'로 쪼개서 점진적으로 스트림하는 것.
- 스트리밍을 함으로써, 너의 전체 페이지 블로킹을 예방할 수 있음. 유저들은 스트리밍이 덜 된 곳 제외하고는 다 볼 수 있고, 상호작용 할 수 있다.
- Next.js에서 스트리밍을 구현하는 방식은 2가지, loading.tsx을 쓰거나, 컴포넌트 레벨에서 Suspense를 통해 더욱 세부적인 제어를 함

### `loading.tsx`파일을 이용해 스트리밍을 구현해보자

- Next.js의 특별한 파일임. React Suspense기반으로 작성됨. 페이지 컨텐츠 로드 중에 폴백 UI를 제공할 수 있게함.
- 근데 이건 페이지 단위..

### Next.js Route Groups가 뭐고, 언제 사용해야 할까요?

- (RouteGroup)/stream_part1, (RouteGroup)/stream_part2 이런식으로 묶을 수 있음.
- 상위 레벨에 loading.tsx가 있으면 하위 요소에도 다! 적용됨, 근데 이런식으로 RouteGroup으로 묶고 아래에 loading.tsx를 넣어주면, 해당 요소에만 스트리밍이 적용됨. ()로 묶인 Route Group은 실제 URL 패턴에는 영향을 끼치지 않으면서 논리적인 그룹을 만들기 위해 사용된다.

### React Suspense boundaries를 두는 올바른 장소

- React Suspense를 이용해 세부적으로 특정 컴포넌트를 스트리밍할 수 있음.
- Suspense는 몇몇 조건이 충족되기까지 어플리케이션 일부 렌더링을 지연시킬 수 있게함.
- 특정 데이터가 필요한 특정 컴포넌트때문에 전체 페이지 로딩이 느려진다 -> 그 부분만 스트리밍적용하면 웹 성능이 좋아지겠지? 그 때 Suspense를 사용하면 된다.

- 그렇다면 Suspense 경계, 실제로 어떻게 판단하면 되냐?

  1. **“느린 컴포넌트를 기다릴 필요가 있나?”** - 특정 섹션이 _항상 느리다_ → 그 부분만 Suspense로 분리하는 것이 합리적. - 예: Dashboard에서 - **Analytics** API가 500~800ms - **프로필 정보**는 50ms
     이런 상황이면 전체 페이지 Suspense 대신 Analytics만 감싸는 게 맞다.

👉 _전체 페이지 Suspense는 모든 데이터가 준비될 때까지 화면이 비어 있음 (loading.tsx만 보임)._

2.  **“불안정한 네트워크로 UI가 튀어 보이는 문제가 생기나?”**

    Suspense를 쪼개면 각 컴포넌트가 준비되는 순서대로 등장한다.
    하지만…

    - 카드 목록 먼저 뜨고
    - 통계 그래프 나중에 나타나고
    - 그 사이에 레이아웃 흔들림 발생
      이런 “UI popping”이 싫으면 **너무 잘게 쪼개지 않는 게 낫다.**

**→ 섹션별로 묶어서 Suspense 2~3개 정도로 나누는 게 보통 실용적.**

3. **“해당 컴포넌트가 직접 데이터를 가져오도록 설계했는가?”**

요즘 권장 패턴:

> **데이터 fetch는 그 데이터를 쓰는 컴포넌트 가까이에 둔다.**
> 그리고 **그 컴포넌트 자체를 Suspense로 감싼다.**

이 방식의 장점:

- 특정 컴포넌트만 느려도 페이지 전체가 끌려 내려가지 않음
- 코드 구조가 feature 단위로 깔끔해짐
- 리팩토링 할 때 안전하고 예측 가능함

단점:

- 위에서 말했듯 UI popping 가능성 증가

4. 결론: Suspense 경계 기준은 “속도 + 중요도 + UX 일관성”

   - **이 섹션이 느리면, 나머지 섹션도 기다리게 해야 하는가?**
   - **이 섹션이 늦게 뜰 때, 사용자 경험이 크게 깨지는가?**
   - **이 섹션은 데이터를 자체적으로 fetch하는가?**
   - **UI popping을 허용할 수 있는가?**
   - **중요 정보부터 먼저 보여줘야 하는가?**

이걸 조합하면 Suspense placement는 자연스럽게 결정됨

## 검색과 페이지네이션 추가

### Next.js API 사용법 : `useSearchParams`, `usePathname`, `useRouter`

- URL search params를 검색 상태관리에 쓰는 이유

  - 1. 북마크 생성 가능, 공유 가능
  - 2. 서버사이드 렌더링 가능
  - 3. 통계 및 추적 가능

- Next.js API
  - `useSearchParams` : 현재 URL의 파라미터에 접근가능.
  - `usePathname` : 현재 URL의 pathname을 읽어줌
  - `useRouter` : 클라이언트 컴포넌트 간 네비게이션 가능하게 함

### URL search parameter를 이용한 검색과 페이지네이션

- client component에서는 useSearchParams로 searchParam받아올 수 있음.
- page component는 props에서 searchParams를 얻어볼 수 있음
  - `const searchParams = await props.searchParams;`

## 데이터 변형(mutating)

### React Server Action이란 무엇이고 data mutate시 사용하는 법

- Server Action이란? : 서버에서 비동기적인 코드를 실행시킬 수 있게 해줌. -> DB 직접 접근, 데이터 소스 직접 접근이 가능하기에 API를 따로 만들 필요 없음
- 보안이 웹 어플리케이션에선 가장 중요함. -> 서버액션은 서버에서 돌기 때문에 보안적으로 강함
- Server Action : ts파일 제일 상단에 'use server'로 서버 함수임을 명시. 그럼 그 파일에서 export되는 모든 함수는 Server Action으로써 판단 됨.
- Server Action은 'Client'컴포넌트에도, 'Server'컴포넌트에도 import해서 사용 가능
- Server Action 파일에 작성된 함수 중 사용되지 않는 함수가 있다면 자동으로 번들에서 빠짐

### 폼들과 React Server Component와 동작하는 법

- React에선 `<form>`엘레멘트의 `action` 어트리뷰트를 서버 액션 실행시키는데 사용합니다.
- action에 Server Action 함수를 등록합니다. -> Server Action 함수는 props로 native FormData를 받습니다.
- **Server Action은 Next.js의 캐싱과도 깊게 연관되어 있습니다. 폼이 Server Action을 통해 제출되면, 데이터 변형하는데 액션을 쓸 수 있을 뿐만 아니라, `revalidatePath`나 `revalidateTag`같은 API를 이용해 연관된 캐시를 revalidate할 수 있습니다.**

### Native `FormData`객체 사용하는 베스트 프랙티스(밸리데이션 포함)

### `revalidatePath` API를 통해 클라이언트 캐시를 revalidate하는 법

### 특정 ID와 함께 dynamic route segment 생성하는법
