# msw 세팅과 버전 업그레이드

### MSW

Mock Service Worker. 개발환경에서의 네트워크 요청을 브라우저 단에서 인터셉트에서 사전에 지정한 응답을 주도록 하는 라이브러리

- 기본은 브라우저에서 돌지만 nextjs처럼 ssr 프레임워크에서는 node에서도 동작하도록 별도 설정 추가 가능

MSW로 API가 없어도 요청을 보낼 수 있다.
요청에 대한 응답을 직접 MSW로 만들어줘야함

### 1. package.json 설정

최하단에 아래 내용을 추가한다

```json
  "msw": {
    "workerDirectory": "public"
  }
```

### 2. 설치

```bash
yarn add msw init public/ --save-dev
```

위치를 public으로 지정했기 때문에 아래와 같이 `mockServiceWorker.js` 파일이 생성된다.
![msw-install](msw-install.png)

### API가 있어도 MSW 활용이 가능하다

에러 케이스를 테스트해보고 싶을 때도 사용 가능.
msw가 API 요청에 대한 응답으로 에러를 주도록 설정할 수 있음.
에러 상황을 억지로 만들어낼 필요 없이 서비스 워커의 응답을 에러로 만들면 된다.

> Next에서는 서버에서도 클라이언트에서도 돌아야하는데, 서버쪽에서 목비스워커를 돌리는 방식이 안나왔다.
> 그래서 임시로 노드서버를 함께 돌려줌

### HTTP cookie 세팅

```json
{
  "headers": {
    "Set-cookie": "connect.sid=msw-cookie'HttpOnly;Path=/"
  }
}
```

HTTP, 헤더 공부하기 (백엔드의 코드도 대략 비슷함)

### package.json에 노드서버 시작 스크립트 설정

```json
"mock": "yarn tsx watch ./src/mocks/https.ts"
```

코드 수정하면 알아서 서버가 재시작됨.

# next용 msw 컴포넌트와 .env

언제 MSW를 적용하고 언제 MSW를 적용하지 않을지 판단하는 컴포넌트 생성.

![msw-component](msw-component.png)

`layout.tsx`의 body 바로 아래에 위치시켜주면,
클라이언트 환경에서는 `mockServiceWorker.js` 파일이 요청을 가로챈다.
가로채서 우리가 작성한 http 서버로 보냄.

### 환경변수 세팅

NEXT_PUBLIC_API_MOCKING=enabled

앞에 NEXT_PUBLIC을 붙이면 브라우저에서 접근 가능한 환경변수.
없는 환경변수는 서버에서만 접근 가능하고 브라우저에서 접근 불가.

브라우저가 접근 가능한 환경변수는 개발자도구에서 모든 사람이 접근 가능하다.

### 브라우저 환경 보장

```ts
if (typeof window !== "undefined") {
  if (process.env.NEXT_PUBLIC_API_MOCKING === "enabled") {
    require("@/mocks/browser");
  }
}
```

window가 있는 경우 === 클라이언트 환경

# 서버 컴포넌트에서 Server Actions 사용하기

우선 훅을 모두 없애야한다. 그럼 동작을 어떻게 하는가?
<리액트는 잊고 html과 javascript에서 어떻게 동작했는지 생각하기>

onClick 액션은 클라이언트 컴포넌트로 따로 분리.

## 서버 컴포넌트 사용

클라이언트 컴포넌트 안에서 서버컴포넌트를 사용할 때에는 "use server"를 붙여주기.
(프론트엔드 서버) 그럼 이건 서버컴포넌트가 되어서 내부 코드는 브라우저에 노출되지 않는다.

- 기존의 단계 : 브라우저-서버-DB
- 현재의 단계 : 브라우저-DB

여기서 바로 DB 접속해서 데이터를 가져올 수 있다.

### 쿠키 - credentials 설정이 왜 필요한가?

쿠키가 있어야만 로그인 유무를 알 수 있다.
하지만 브라우저가 제공하는 요청 API는 브라우저의 쿠키와 같은 인증과 관련된 데이터를 요청 데이터, 또는 응답 데이터에 담지 않는다.
요청과 응답에 쿠키를 포하하고 싶다면 `withCredentials` 옵션을 사용.

현재 클라이언트는 localhost:3000, 서버는 localhost:9090.
_이때 CORS 에러가 발생한다_
클라이언트나 서버나 둘다 Credentials를 true로 설정해야한다.

표준 CORS요청은 기본적으로 쿠키를 설정하거나 보낼 수 없다.
프론트에서 ajax 요청할 때, withCredentials부분을 true로 해서 수동으로 CORS 요청에 쿠키값을 넣어줘야 한다.
마찬가지로 서버도 응답헤더에 Access-Control-Allow-Credentials를 true로 설정해야 한다.

#### 클라이언트에서는

- axios일 경우

  - withCredentials 옵션 부분을 axios 전역 설정으로 처리
  - 또는 axios 요청 메소드의 옵션 인자로 넣어 전달

  ```js
  axios
    .post("url", { username: username }, { withCredentials: true })
    .then((response) => {
      console.log(response);
      console.log(response.data);
    });
  ```

- fetch일 경우

```js
fetch("url", {
  method: "POST",
  credentials: "include", // 클라이언트와 서버가 통신할때 쿠키 값을 공유하겠다는 설정
});
```

#### 서버에서는

서버에서도 설정이 필요하다.
만일 서버에서 별도의 처리 없이 클라이언트만 withCredentials으로 서버에 cors 요청하게 되면 모두 거부된다.

- 서버 node일 경우
  서버에 response 헤더(Header) 값으로 Access-Control 설정을 해준다.

  ```js
  response.setHeader("Access-Control-Allow-origin", "*"); // 모든 출처(orogin)을 허용
  response.setHeader(
    "Access-Control-Allow-Methods",
    "GET, POST, OPTIONS, PUT, PATCH, DELETE"
  ); // 모든 HTTP 메서드 허용
  response.setHeader("Access-Control-Allow-Credentials", "true"); // 클라이언트와 서버 간에 쿠키 주고받기 허용
  ```

- 서버 express일 경우

```js
const express = require("express");
const cors = require("cors");

const app = express();

app.use(
  cors({
    origin: "*", // 출처 허용 옵션
    credential: "true", // 사용자 인증이 필요한 리소스(쿠키 ..등) 접근
  })
);
```

### 회원가입 - 서버컴포넌트에서 서버액션

form 데이터로 한번에 받으려면 id 속성 외의 name 속성이 필요하다.
form 데이터의 name으로 데이터를 직접 가져오기 때문

```tsx
const submit = async (formData: FormData) => {
  "use server";
  if (!formData.get("id")) {
    return { message: "no_id" };
  }
  if (!formData.get("name")) {
    return { message: "no_name" };
  }
  if (!formData.get("password")) {
    return { message: "no_password" };
  }
  if (!formData.get("image")) {
    return { message: "no_image" };
  }
  let shouldRedirect = false;

  try {
    const response = await fetch(
      `${process.env.NEXT_PUBLIC_BASE_URL}/api/users`,
      {
        method: "post",
        body: formData,
        credentials: "include",
      }
    );
    if (response.status === 403) {
      return {
        message: "user_exists",
      };
    }
    console.log("response", response);
    console.log(await response.json());
    shouldRedirect = true;
  } catch (err) {
    console.log("errr", err);
  }
  if (shouldRedirect) {
    redirect("/home");
  }
};
```

```tsx
<form action={submit}>
  <div className={style.modalBody}>
    <div className={style.inputDiv}>
      <label className={style.inputLabel} htmlFor="id">
        아이디
      </label>
      <input
        id="id"
        name="id"
        className={style.input}
        type="text"
        placeholder=""
        required
      />
    </div>
    <div className={style.inputDiv}>
      <label className={style.inputLabel} htmlFor="name">
        닉네임
      </label>
      <input
        id="name"
        name="name"
        className={style.input}
        type="text"
        placeholder=""
        required
      />
    </div>
    <div className={style.inputDiv}>
      <label className={style.inputLabel} htmlFor="password">
        비밀번호
      </label>
      <input
        id="password"
        name="password"
        className={style.input}
        type="password"
        placeholder=""
        required
      />
    </div>
    <div className={style.inputDiv}>
      <label className={style.inputLabel} htmlFor="image">
        프로필
      </label>
      <input
        id="image"
        name="image"
        className={style.input}
        type="file"
        accept="image/*"
        required
      />
    </div>
  </div>
  <div className={style.modalFooter}>
    <button className={style.actionButton}>가입하기</button>
  </div>
</form>
```

현재는 서버 액션을 위해 서버 컴포넌트인 상태.
오류 메시지를 화면에 표시하기 위해 Signup 모달은 클라이언트 컴포넌트가 되어야한다.

\*참고
next/navigation의 `redirect()`는 try-catch에서 사용할 수 없다.

# 서버 컴포넌트에서 Server Actions 사용하기

클라이언트 컴포넌트에서도 서버액션을 쓸 수 있다.

react가 제공하는 useFormState와 useFormStatus를 사용.

```tsx
const { state, formAction } = useFormState(submit, { message: null });
// 전달한 액션 submit를 state에서 관리한다
// 초기 state를 지정
const { pending } = useFormStatus();
```

# 미들웨어, API 라우트, catch-all 라우트

https://authjs.dev/?_gl=1*1fc55c1*_gcl_au*MTYxNTY0ODk5OS4xNzE2ODYyNTI1Ljc2MDE2MTU1LjE3MTY4NjI2MjcuMTcxNjg2MjYyNg

## `auth.js`

next, svelt, soild 제공
provider를 제공해서 카카오, 네이버, 구글, 애플 로그인 등으로 간편로그인을 구현할 수 있음!

## Middleware

: 페이지 접근권한 조절이 매우 쉽다

[Next middleware 공식문서](https://nextjs.org/docs/app/building-your-application/routing/middleware)

next가 제공하는 기능.
페이지별 접근 권한을 설정할 수 있다

```ts
import { auth } from "./auth";
import { NextResponse } from "next/server";

export async function middleware() {
  const session = await auth();
  if (!session) {
    return NextResponse.redirect("http://localhost:3000/i/flow/login");
  }
}
// See "Matching Paths" below to learn more
export const config = {
  matcher: ["/compose/tweet", "/home", "/explore", "/messages", "/search"],
};
```

이 matcher에 해당되는 라우트는 페이지 랜더링되기 전에 `middleware` 함수가 먼저 작동한다.
로그인하지 않았으면 로그인페이지로 리다이렉트시킬 수 있음.

## API route

`route.ts` 이 파일이 API가 되는 것.

백엔드 없이도 next에서 모든걸 처리할 수 있다.
프론트 서버에서 DB와 연결, DB 처리, DB에서 데이터 가져오기, 회원가입하기 등등 모든 걸 다 처리할 수 있다.

> 그럼 프론트와 백엔드를 왜 분리하는가?
> 백엔드만 요청이 많이 가고 프론트엔 요청 안가는 기능 -> 백엔드 서버만 여러대로. 반대도 마찬가지
> 하지만 프론트와 백엔드를 한 서버에 몰아넣으면 한쪽에 요청이 많을 때 둘 다 늘려줘야한다.
> 그래서 보통 서버를 기능별로 나누는 것. 그래야 한쪽만 여러 대로 늘릴 수 있으니까

```ts
export { GET, POST } from "@/auth";

// export function GET() {}
// export function POST() {}
```

우리가 직접 만들 수도 있지만, auth에서 만든 NextAuth의 GET과 POST를 그대로 사용하도록 함.

nextauth와 관련된 API 주소 요청은 전부 다 여기로 들어온다.

## catch-all route

[Next catch-all segments 공식문서](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes#catch-all-segments)

[...이름]

GET /api/auth/a
GET /api/auth/b
GET /api/auth/a/b
GET /api/auth/a/b/c

어떤 주소든 다 들어갈 수 있다

모두 다 잡아서 이 파일로 들어옴.

# next-auth로 로그인하기

auth에 아래와 같이 정의

```ts
export const { handlers, auth } = NextAuth({
  pages: {
    signIn: "/i/flow/login",
    newUser: "/i/flow/signup",
    // next-auth가 기본적으로 제공하는 화면을 쓰지 않고 커스텀화면 사용 지정.
  },
  providers: [
    CredentialsProvider({
      async authorize(credentials) {
        const authResponse = await fetch(
          `${process.env.NEXT_PUBLIC_BASE_URL}}/api/login`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              id: credentials.username,
              password: credentials.password,
              // 우리는 `handler.ts`에서 id와 password로 정의했지만,
              // next-auth에서는 username과 password로 고정되어있어서 변환.
            }),
          }
        );

        if (!authResponse.ok) {
          return null;
        }

        const user = await authResponse.json();
        console.log("user", user);

        return {
          email: user.id,
          name: user.nickname,
          image: user.image,
          ...user,
        };
        // next-auth에는 누가 로그인했는지 가져올 수 있음
      },
    }),
  ],
});
```

## LoginModal

서버에서는 `import { signIn } from "next-auth/react";` 사용,
클라이언트에서는 `import { signIn } from "@/auth";` 사용.

사용 방법은 같다

```ts
const onSubmit: FormEventHandler<HTMLFormElement> = async (e) => {
  e.preventDefault();
  setMessage("");
  try {
    await signIn("credentials", {
      username: id,
      password,
      redirect: false, // 여기서 true로 하면 서버쪽에서 리다이렉트됨.
    });
    router.replace("/home"); // 클라이언트에서 라우터로 replace함
  } catch (err) {
    console.error(err);
    setMessage("아이디와 비밀번호가 일치하지 않습니다.");
  }
};
```

### kakao 로그인을 추가하고 싶은 경우

```ts
await signIn("kakao", {
  username: id,
  password,
  redirect: false,
});
```

auth의 CredentialsProvider 아래에 추가하면 됨.

# 로그아웃 & 로그인 여부에 따라 화면 다르게 하기

### 세션정보 불러오기 함수

클라이언트에서 `useSession`

```tsx
import { useSession } from "next-auth/react";
const { data: me } = useSession();
```

서버에서

```tsx
import { auth } from "@/auth";
const session = await auth();
```
