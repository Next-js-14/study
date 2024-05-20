### 조건부 쿼리 & 쿼리 재사용하기
- 로그인 한 사용자에게만 표시되게 하려면 enabled: !!session?.user 로 표시하면 됨

### 서버사이드 렌더링을 적용하는 기준
- 검색페이지에 노출되어야 하는 정보에 적용(유저 페이지, 게시글 상세페이지 등)
- 기본적으로 돼 있기 때문에 개발자가 끄는 경우가 많음
- 서버쪽에서 쿼리를 해온 것을 dehydrate state로 받으면 됨. Hydration Boundary 로 감싸줌

### 인피니트 스크롤링
- lastID에 따라서 다음 게시글 5개를 불러옴
- 프론트에서 5개씩 계속 데이터를 더 가져올 거고 그때 쿼리 스트링으로 lastID(커서라고도 많이 표현함) 를 전달해줌. 그러면 서버에서 lastID 다음 5개의 게시글을 돌려줌
- 프론트에서는 처음에 기본값을 0으로 해서 보냄
- const cursor = parseInt(url.searchParams.get('cursor'))||0
- return문에서 postID: cursor + 1 이런 식으로 5까지 적어줌
- Home에서 prefetchQuery를 prefetchInfiniteQuery 바꿔주고 initialPageParam:0 (커서값)을 넣어야 함

### Intersection Observer
- 페이지를 가장 밑으로 내렸을 때 다음 페이지를 불러오는 함수를 호출해야 하기 위해 사용
- 화면 밑바닥에 가상태그를 두고 스크롤이 끝에 닿으면 이벤트를 호출하는 것임
- fetchNextPage, hasNextPage
- 다섯개의 배수로 불러오므로 5의 배수일 때는 한 페이지를 더 불러와야 함
- Height 50 정도 영역을 깔아주고 이게 보이기 시작하면 fetchNextPage 함수를 호출
- React intersection observer에는 useInView라는 훅이 있음. 훅에는 ref, inView가 들어있고, 옵션으로 threshold가 있음. 
- threshold는 div가 보이고 나서 몇 픽셀 후에 호출할 건지를 정하는 것임. 보이자마자 바로 호출하려면 0으로 설정
- delay옵션도 있는데 화면에 보이는 순간 보이게 하려면 0으로 하면 됨
- 호출은 useEffect 훅을 사용하고 inView가 true가 되면 작동하도록 하면 됨. 화면에 보이는 순간 fetchNextPage 함수를 호출하면 됨

### Suspense로 streaming
- 폴더 내에 loading.tsx, error.tsx 를 만들 수 있는데, 리액트의 기본기능을 활용하는 것임 -> suspense, loading boundary
- Suspense는 리액트17에서 추가됐는데, 잠깐 중단하다, 유예하다의 의미임 : 페이지에 로딩이 될때까지 잠깐 멈추고 로딩을 보여주다가, 로딩이 되면 페이지를 보여주는 것임
- 에러 핸들링에서는 error boundary를 많이 씀. 페이지에서 에러가 발생했을 때 Error컴포넌트를 띄움. 
- 로딩이랑 에러 같이 쓰고 있으면 에러 바운더리 안에 susense가 들어가고 그 안에 페이지가 들어가는 계층구조가 만들어짐.
- 로딩이 된 것을 판단하는 방법: 서버에서 데이터 가져오기하는 경우 페치하는 것을 기다리는 경우 / 리액트 lazy가 적용되었을 때 / use라는 훅을 사용하는 경우 (promise가 resolve되기까지 suspense가 기다려줌)
- 처음 접속할 때는 페이지를 전부 그려서 보내주므로 로딩이 필요 없음. 다른 페이지에 들어갈 때는 로딩창이 뜸. 
- Suspense를 쓰면 특정 부분만 로딩을 지연시킬 수 있음. 스트리밍 방식이기 때문에 여러번 클라이언트로 데이터를 보낼 수 있음. 
- 레이아웃 같은 부분은 어차피 넥스트에서 먼저 알아서 보내주기 때문에, 상단 탭을 먼저 분리하는 것도 가능(TabDecider). 로딩이 안 필요한 부분만 먼저 렌더링함.
- useSuspenseQuery, useSuspenseInfiniteQuery를 사용하는 것을 추천
