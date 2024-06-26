## Next 14

- Next.js 공식문서를 보면 app router / pages router 두 가지로 구성되어 있다.

### 서버 컴포넌트
next 서버에서 리액트를 미리 렌더링해서 프론트 혹은 브라우저, 클라언트로 데이터를 보내줄 때 완성된 HTML을 미리 보내주는 것

### 장점
HTML 로딩 시간, JS 용량 감소
<br/>
### 단점
Next 서버 자체의 부담 증가 → 이에 따라 ‘캐시’ 적극 활용
<br/>

### public 폴더
- 넥스트 서버에서 누구나 접근할 수 있게 서비스 해준다.
- 모든 사람이 접근할 수 있는 사이트에서 쓰일 이미지 등을 넣는 것이 좋다.

### src 폴더
- 주로 루트 아래에 src, app이 별도로 있는 것이 원칙이지만 `src/app` 구조는 타입스크립트를 한번에 처리하기에 용이하다.

### app 폴더
- 주소와 관련된 파일이 들어가 있다.

### next.config.js
- 넥스트 설정 파일

### tsconfig.json
- 타입스크립트 설정 파일

**path alias**
경로에 대한 별칭
‘../../app/layout’ ⇒ ‘@/app/layout’ 으로 변경이 가능

### Layout
- 레이아웃의 경우 애플리케이션 내에서 페이지 이동 시 리렌더링 X
- 페이지 이동 시에는 ‘페이지’만 리렌더링 되는데, 만약 레이아웃도 리렌더링이 되게 하고 싶다면 `template.tsx` 를 사용하면 된다.

### <Image>
`<img>` 가 아닌 `<Image>` 를 사용하게 되면 import 한 이미지를 Next에서 자동으로 최적화해준다.

###  Server Component,  Client Component
- Next.js에서는 기본적으로 모두 서버 컴포넌트
- Client Component로 바꾸려면 상단에 ‘use client’
- Server Component는 전부 데이터와 관련이 있음

### Parallel Route
- Parallel Route를 사용하면 동일한 레이아웃에서 하나 이상의 페이지를 동시에 또는 조건부로 렌더링할 수 있다.
- 배경에 화면이 남아있고 모달을 띄운다.
- 기존 모달과의 차이점 : 기존 모달과의 차이점은 주소가 바뀌냐 안 바뀌느냐이 차이다. 패러렐 라우트는 동시에 띄워진 페이지의 주소가 각각 다르고, 기존 모달은 주소 변경 없이 모달이 동작한다.
