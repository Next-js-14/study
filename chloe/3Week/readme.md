### 컴포즈트윗 부분
- 패러랠 라우트 할 때는 항상 layout에 모달 자리를 마련해야 함

### usePathname
- /explore일 때 null을 반환하도록 usePathname 훅을 사용
- 훅이기 때문에 use client 
* 라디오 버튼 - 버튼 위에 div 올려놓은 형태, ref로 연결해주면 됨

### 폴더구조
- 컴포넌트의 특성이 여기서만 쓰이는지, 아니면 둘 이상 공유되는지, 아니면 앱 전체에 걸쳐서 임포트되는지에 따라 폴더구조 결정

### useSearchParams
- 주소창에 정보를 넣어놓는 것이 search params
- q, src, vertical이 있으며, search params의 props로 받아오면 됨. 필수가 아니라 옵셔널임
  - q="악의 평범성"&src=trend_click&vertical=trends
  - 이런 것을 페이지에 대한 정보로 가져올 수 있음
- Tab 누를때 주소가 바뀌는 것은 useRouter를 사용하면 됨
- q값을 얻어와야 하는데 방법이 두개가 있음.
  - 1) 부모인 search페이지에서 props로 searchParams를 받아오기 때문에 그대로 Tab에 넘겨주는 것
  - 2) useSearchParams 훅을 이용해서 searchParams.get('q')
- searchParams.toString()은 지금 있는 거 그대로 다 쓰고 하나더 추가할 경우에 사용(ex. &f=live를 추가)

### 서버컴포넌트와 클라이언트 컴포넌트를 같이 쓰는 방법
- 부모는 클라이언트, 자식은 서버 컴포넌트인 경우 => 서버컴포넌트를 클라이언트 컴포넌트의 children으로 넣으면 됨
- 서버 컴포넌트를 임포트해서 쓰면 안되고 children이나 props로 넘겨주면 서버컴포넌트가 제대로 작동함
- 만약 서버컴포넌트를 임포트하는 경우는 클라이언트 컴포넌트로 성격이 바뀌어버림.

### 이벤트 캡처링
- 클릭이벤트랑 A태그랑 겹치는 등의 경우에 활용 가능

### Move module members 기능
- export 문에 있는 기능을 통째로 옮겨야 할 때 사용
- 이 프로젝트에 있는 임포트를 전부 바꿔주면서 옮겨주기 때문에 편리

### faker 라이브러리
- npm i @fakerjs/faker로 설치
- 더미데이커를 쉽게 만들어주는 라이브러리
- 디폴트 임포트가 아닌 네임드 임포트로 사용
- urlLoremFlicker하면 매번 랜덤한 이미지가 뜸

### 이미지 & PostImages 컴포넌트
- Math.random()을 사용해서 반반 확률로 이미지가 있거나 없거나 하게 할 수 있음
- 슬러그들의 값을 params로 가져올 수 있음
- 이미지가 몇 개 있느냐에 따라서 다르게 디스플레이하기 위해 cx를 사용함

### MSW (Mock Service Worker)
- AP가 아직 만들어지지 않았을 때 사용, DB를 다루지는 않고 이런 요청을 보내면 이런 응답이 올 것이라고 미리 코딩해놓는 것임
- npx msw init public/ --save 명령어 입력하면 public 폴더에 파일이 생성됨. 서버에 보내는 요청을 msw가 가로챔
- 실제 백엔드개발자가 있을 때도 개발환경에서 억지로 에러를 내야 한다거나 로그인안해도 되도록 할 때 사용할 수 있음
- next에서는 msw 사용이 애매한게 서버에서도 돌고 클라이언트에서도 돌아야 하기 때문에 서버쪽에서는 msw를 자연스럽게 돌리는 방식이 아직 안나옴. 여기서는 임시로 노드 서버를 활용
- 최근에 1버전에서 2버전으로 업데이트됨(공식문서 확인)
  *http, 쿠키에 대한 공부 필요
- msw에서 쓸 것은 handlers.js에 넣어줌
- package.json에 mock을 실행하는 명령어를 추가하고, 터미널에서 npm run mock을 입력하면 됨 
- next 앱에서 언제 msw를 적용하고 적용하지 않을지 판단하기 위해 App 폴더 내에 Componenet>MSWComponent.jsx를 만들고, Layout의 바디태그 안에 해당 컴포넌트를 넣음
- 환경설정에 NEXT_PUBLIC_API_MOCKING=enabled 를 추가
- NEXT_PUBLIC이 있으면 브라우저에서 접근가능하고(개발자 도구에서 유출이 가능한 상태) 그게 없으면 서버에서만 접근 가능(유출이 불가능한 상태)

### .env & .env.local
- 배포환경용은 .env, 개발환경용은 .env.local
- 개발을 하는 동안에는 둘다 실행됨

### 서버 컴포넌트에서 Server Action 사용
- 14버전부터 쓸 수 있음
- 여기서는 SignupModal.jsx에서 submit을 server actions 로 사용
  - 매개변수로는 FormData를 받음
  - "use server"를 적으면 됨
  - 브라우저에 노출이 안되므로 키값을 사용해도 됨
  - 기존처럼 브라우저->서버->DB 이런 접근을 하지 않고 서버->바로 DB접속해서 데이터를 가져올 수 있음
  - 이 경우 프론트 서버라서 백엔드 서버로 한번더 요청을 보냄
  - credentials: 'include' 있어야 쿠키가 전달이 됨(이미 로그인한 사람이 회원가입하는 경우 쿠키가 있어야 로그인했는지 알아낼 수 있음)
  - 검증 메시지는 클라이언트에서 보여줘야 하는데, 이미 서버 컴포넌트이기 때문에, 다시 클라이언트 컴포넌트로 바꾸면서 서버액션은 같이 쓰는 형태로 바꿔야 함
  * redirect는 try-catch문으로 사용하면 안됨

### 클라이언트 컴포넌트에서 Server Action 사용
- 검증 메시지를 클라이언트에서 표시해줘야 함
- useFormState(Form에서 state 사용), useFormStatus(Form에서 처리하고 있는 정보 접근) 훅을 활용함
  - FormStatus에서 pending을 활용
- showMessage라는 function을 만들어서 검증 메시지를 화면에 표시

### Auth.js(NextAuth)
- 5버전으로 업데이트됨
- 로그인프로바이더에 다양한 로그인을 지원(kakao, naver 포함)
- 아이디 비번 로그인은 CredentialsProvider
- 로그인에 따른 쿠키관리도 지원함
- 앱폴더와 같은 레벨에 auth.js와 middleware.js를 만들어줌
- npm i next-auth@5 @auth/core 로 설치가능
- middleware는 next에서 공식지원하는 기능. middleware의 config부분에는 로그인에 따른 페이지 접근권한 설저이 가능함. 페이지 렌더링되기 전에 실행됨
- auth함수를 호출하면 로그인했는지 여부를 알아낼 수 있음. 
- signin 함수는 로그인하는 용도
- handlers는 API route임

### API route
- 브라우저처럼 실제 주소가 됨
- 앱 폴더 아래에 api > auth > [..nextauth] > route.js 를 생성
  - ...의 의미는 cath-all 라우트임. 어떤 주소든 다 들어갈 수 있다는 의미임
- 작은 서비스의 경우에는 프론트, 백엔드 서버 따로두면 불편함. 그냥 프론트 서버 하나에서 api route, route.js를 만들어서 처리해도 됨
- 서비스 규모가 커지면 프론트 서버, 백엔드 서버를 나누는 게 좋음
  - 한 쪽에만 요청이 많이 올 때 서버를 늘려주면 되기 때문
- handlers에 만들어놓은 GET, POST를 route.js에서 사용하면 됨
- .env에 쿠키를 암호화하는 AUTH_SECRET을 추가
- nextAuth에서 기본적으로 제공하는 페이지 대신 우리 페이지를 추가해야 함

### useSession
- 로그인을 했는지 안했는지 확인하는 용도
- Provider로 감싸주어야 함
- 로그인 쿠키를 훔쳐가는 CSRF공격을 nextAuth에서 토큰으로 막아줌
- 네트워크탭에서 session에 대한 응답을 확인가능