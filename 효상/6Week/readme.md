# 6Week

## 1. 백엔드 서버 세팅하기

- PostgreSQL

### PostgreSQL과 MySQL의 차이점

- MySQL은 읽기 전용 명령을 관리하는 데 주로 사용되며, PostgreSQL는 읽기-쓰기 작업, 대규모 데이터 세트 및 복잡한 쿼리를 관리하는 데 주로 사용된다.
- MySQL은 PostgreSQL보다 기능이 적지만, 읽기 전용 쿼리에서 더 가볍고 안정적이며 빠른 처리 속도를 유지할 수 있다.
- MySQL은 관계형 데이터베이스 관리 시스템(RDBMS), PostgreSQL은 객체 관계형 데이터베이스 관리 시스템(ORDBMS)이다.

### Oracle VS PostgreSQL

- Oracle은 성능이 좋고 속도가 빠른 대신 유지비용이 많이 든다.
- PostgreSQL은 유지비용이 적게 드는 대신 Oracle보다는 성능이 좋지 않다.
- 요새 유지비용 문제로 Oracle > PostgreSQL로 많이 전환하는 추세
- 문법이 조금씩 다르다.

## 2. 로그인과 회원가입 실제로 하기

### 회원가입 후 자동 로그인이 될 때 로그아웃 버튼이 나타나지 않는 현상

- 현재 로그인 이후의 코드들을 확인해보면, server action으로 만들어둔 auth() 의 반환 값을 할당한 session 에는 값이 존재하지만
- LogoutButton 컴포넌트에서 useSession() 으로 가져온 반환 값은 null 인 상태이다.
- 따라서 useSession() 훅을 통해서 값을 가져오지 않고, 레이아웃에서 LogoutButton으로 props를 넘겨주도록 코드를 수정한다.

### 내가 주로 사용했던 방법

- 결국 SPA이므로, 로그인 이후에 사용자의 상태를 업데이트 해주기가 어렵다. (재 렌더링이 일어나지 않기 때문에) 따라서 이러한 문제를 해결하는 방법은 재렌더링을 일으키는 방법
- 1. window.location.href('/') : ''안에 있는 페이지로 이동을 시켜준다 (a 태그의 href와 같은 맥락) 따라서 해당 페이지로 이동 시킨 후에는 재렌더링이 일어나므로 세션의 쿠키 유무를 읽을 수 있음
- 2. 데이터를 비우고 해당 페이지의 axios 통신을 다시 요청 : getData() 등의 함수로 해당 페이지의 axios 통신을 담고 로그인 , 로그아웃 , 기타 상태값의 리셋이 필요한 경우 다시 통신하여 데이터를 가져온다 > 현재 프로젝트에서 사용중 (reload() 함수 안에 넣어서 사용하는 중)

## 3. 서버 쿠키 공유하기

### 프론트에서는 session-token으로 로그인 여부를 확인할 수 있고, 백엔드에서는 connect.sid로 로그인 여부를 확인할 수 있다.

- 해당 부분이 NEXT.JS의 강점인거 같은데 다른 직원분에게 듣기로는 NEXT.JS에서 ONE SERVER로 진행 할 경우 프론트단과 백단의 인터페이스 공유가 가능하다고 들었음
- 특히 같은 JS 언어로 되어 있는 프론트와 백단의 경우 미리 설정해 놓은 유틸이나 인터페이스 등으로 데이터 타입 , 사용하는 함수등을 자유롭게 공유 가능

ex) TimeStamp 타입을 원하는 String 형식 ('YYYYMMDD' 등) 로 바꾸는 로직
ex2) 주고 받는 데이터의 Interface

```ruby

interface people {
  id: number;
  name: string;
  age: number;
} // 해당 interface를 front back 모두 공유

```

ex3) session 정보 읽기 등등

- 기존의 JS + JAVA의 경우에는 서로 언어가 달라서 따로 선언해서 다르게 써야 했음 (현재 그러는 중)

## 4. useMutation 사용하기

### useMutation의 사용 이유

- 데이터 상태 관리가 가능하다 (loading, error, success)
- submit 함수에서 처리해줬던 try-catch를 react query에 위임하고, 요청 값만 return 해주면 된다.

### 기존의 submit 함수

```ruby

const onSubmit: FormEventHandler = async (e) => {
    e.preventDefault();

    const formData = new FormData();
    formData.append("content", content);
    imagePreview.forEach((image) => {
      image && formData.append("images", image.file);
    });

    try {
      const response = await fetch(
        `${process.env.NEXT_PUBLIC_BASE_URL}/api/posts`,
        {
          method: "post",
          credentials: "include",
          body: formData,
        }
      );

      if (response.status === 201) {
        setContent("");
        setImagePreview([]);

        const newPost = await response.json();

        // 추천 탭 게시글에 등록
        queryClient.setQueryData(
          ["posts", "recommends"],
          (prevData: { pages: Post[][] }) => {
            const shallow = { ...prevData, pages: [...prevData.pages] }; // 구조분해할당
            shallow.pages[0] = [...shallow.pages[0]];
            shallow.pages[0].unshift(newPost);
            return shallow;
          }
        );

        // 팔로우 탭 게시글에 등록
        queryClient.setQueryData(
          ["posts", "followings"],
          (prevData: { pages: Post[][] }) => {
            const shallow = { ...prevData, pages: [...prevData.pages] };
            shallow.pages[0] = [...shallow.pages[0]];
            shallow.pages[0].unshift(newPost);
            return shallow;
          }
        );
      }
    } catch (err) {
      alert("업로드 중 에러가 발생했습니다.");

      // 사실 alert 보다는 공통 에러 모달을 많이 사용하는 편
    }
  };

```

- try catch 문으로 성공상태와 실패상태를 관리할 수 있음
- 구조분해할당으로 데이터 쉽게 저장 가능
- 근데 NEXT 하다보면 점점 .NET 스럽게 느껴진다.

### try-catch 위임

```ruby

const mutation = useMutation({
  mutationFn: async (e: FormEvent) => {
    e.preventDefault();

    const formData = new FormData();
    formData.append("content", content);
    imagePreview.forEach((image) => {
      image && formData.append("images", image.file);
    });

    return fetch(`${process.env.NEXT_PUBLIC_BASE_URL}/api/posts`, {
      method: "post",
      credentials: "include",
      body: formData,
    });
  },
  async onSuccess(response) {
  // 요청 성공 시 실행 (try)
    const newPost = await response.json();

    setContent("");
    setImagePreview([]);

    if (queryClient.getQueryData(["posts", "recommends"])) {
      queryClient.setQueryData(
        ["posts", "recommends"],
        (prevData: { pages: Post[][] }) => {
          const shallow = {
            ...prevData,
            pages: [...prevData.pages],
          };
          shallow.pages[0] = [...shallow.pages[0]];
          shallow.pages[0].unshift(newPost);
          return shallow;
        }
      );
    }
    if (queryClient.getQueryData(["posts", "followings"])) {
      queryClient.setQueryData(
        ["posts", "followings"],
        (prevData: { pages: Post[][] }) => {
          const shallow = {
            ...prevData,
            pages: [...prevData.pages],
          };
          shallow.pages[0] = [...shallow.pages[0]];
          shallow.pages[0].unshift(newPost);
          return shallow;
        }
      );
    }
  },
  onError() {
	// 요청 실패 시 실행 (catch)
    alert("업로드 중 에러가 발생했습니다.");
  },
});

return (
		// onSubmit props에 mutate 함수 전달
    <form className={style.postForm} onSubmit={mutation.mutate}>
// ...

```

- 솔직히 근데 try catch랑 큰 차이점은 잘...
- 중요한 부분 : async await 로 비동기처리를 해야 post요청의 response를 받을 수 있음