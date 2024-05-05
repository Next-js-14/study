### React Query SSR설정
- 서버에서 데이터 fetching하는 대표적인 라이브러리인 React Query로 데이터 넘겨바은 후 무한 스크롤링
- npm i @tanstack/react-query@5
- npm i @tanstack/react-query-devtool@5 -D
- RQProvider 파일을 만드는데 프로바이더를 설정해두면 그 프로바이더 안에 들어있는 컴포넌트는 리액트 쿼리로 데이터 공유가 됨
- RQProvider 바깥에 있는 애들은 데이터에 접근을 못하기 때문에 감쌀 때 주의해야 함
- Home의 page.tsx에서 적용할 때, 서버에서 불러온 데이터를 클라이언트의 react query가 물려받는다(hydrate한다)
  - Hydrate 라는 것은 서버에서 온 데이터를 클라이언트에서 그대로 형식 맞춰서 물려받는 것임. Hydration Boundary를 설정해서 감싸주고, 데이터를 불러오고 난 이후의 dehydration 상태관리를 해야 함
  - queryKey와 queryFn은 세트임. 이런 키(배열 형태)를 가지고 있을 때는 항상 이 함수를 실행하라는 뜻임
- 서버 컴포넌트에서 받아온 데이터는 자동으로 저장이 됨. 저장을 안하려면 cache: 'no-store'하면 됨
- 서버 컴포넌트의 next tag는 나중에 revalidateTag에서 tag와 관련된 것을 캐시 초기화를 하기 위한 것이고, revalidate Path는 페이지에 있는 것들을 새로 고침하기 위한 것임

### React query DevTool
- 어떤 데이터를 실제로 불러오는지 확인 가능
- fresh, fetching, paused, stale, inactive 등 state도 알아두면 좋음
  - fresh: 서버에서 데이터를 가져온 것. 최신 데이터이고 굳이 업데이트할 필요 없다는 의미. 캐시에 들어있는 것 쓰면 됨. 모든 데이터는 기본적으로 fresh가 아니라는 기본 가정이므로 언제까지 fresh인지는 직접 설정을 해줘야 함
  - fetching: 데이터를 가져오는 것. 순간적이라서 거의 못봄
  - stale: 기회가 되면 데이터를 새로 가져오라는 의미. refetchOnWindowFocus(탭 전환했다가 돌아온 경우), refetchOnMount(컴포넌트가 react에 마운트됐을 때 다시 가져옴), refetchOnReconnect(인터넷 연결이 끊겼다가 다시 된 경우), retry(데이터 가져오기 실패한 경우 재시도) 옵션이 있음.
    - useQuery에서 덮어쓰기로 재설정 가능
    - StaleTime:0 이면 0초 뒤에 fresh에서 stale로 간다는 의미, 60000이면 1분이 지나면 서버에서 데이터를 새로 가져온다는 의미
    - cacheTime: gcTime(garbage Collector Time)으로 이름이 변경됨, 기본적으로 5분으로 설정돼있음, 키값 같은 것들을 메모리에 저장을 하는데, 안쓰는 데이터들을 정리해주는 것, inactive인 데이터들이 5분 뒤에는 사라짐. gcTime이 staleTime보다 길어야 함.
  - paused: 데이터를 가져오는 것을 잠깐 멈추는 경우가 있는데 그런 기능 갯수
- Action 섹션: 데이터를 새로 가져올 수 있음
  - refetch: 새로 가져오는 기능
  - invalidate: refetch와 거의 비슷하나 observer가 1이 될때(현재 화면에서 데이터를 쓰고 있을때만) 다시 가져오는 좀더 효율적인 기능
  - reset: 초기데이터가 있는 경우에 초기데이터로 리셋이 됨
  - restore loding: 로딩상태 제대로 표시되나 확인용

### Redux대신 React query를 쓰는 이유
- Redux를 써도 서버에서 데이터를 받아와서 state에 넣어둘 수 있음
- React query의 핵심기능은 서버의 데이터를 가져오는 것, Redux의 핵심기능은 데이터를 컴포넌트 간에 공유하는 것
- React query는 데이터를 가져오면서 캐싱을 잘해줌: 사용자의 트래픽 관리를 잘해야 하는데, 트래픽 관리를 하려면 데이터를 최대한 캐싱을 해두는게 좋음. 사용자가 빠르게 컨텐츠를 접하는 측면에서도 캐싱을 많이 하는게 좋음. react query가 이것을 관리해줌
- 이번에 tanstack query로 이름이 바뀜
- React query에서도 데이터를 컴포넌트 간에 공유하는 것이 가능. Query.getClient 같은 걸로 공유함
- React query의 또 다른 장점은 인터페이스를 표준화한 것임. 데이터 가져올 때 로딩, 성공, 실패라는 상태가 있는데, 프론트에서 데이터가져올 때 필요한 게 다 표준화가 돼 있음. 
- 요즘 데이터 가져와서 캐싱하는 것은 react query, 데이터 컴포넌트 간에 공유하는 것은 zustand, contextAPI를 많이 씀.




