# 5Week

## 1. 게시글 상세 페이지, 답글, 포토모달

- cache: 'no-store' 를 사용하지 않을 경우 기본적으로 데이터가 캐싱된다. 이는 next의 tags 를 revalidation (재검증) 하기 전까지는 캐싱이 되어 있다.

### 캐싱이란

- 캐싱은 파일 복사본을 캐시 또는 임시 저장 위치에 저장하여 보다 빠르게 액세스할 수 있도록 하는 프로세스

## 2. 미흡한 부분 구현하기

### searchParams 추가하기

- URLSearchParams()
- URL의 쿼리 문자열을 대상으로 작업할 수 있는 메서드
- set(name, value) : 매개변수(name)에 값(value)를 지정한다.

```ruby

const onChangeFollow = () => {
  const newSearchParams = new URLSearchParams(searchParams); // 선언부
  newSearchParams.set("pf", "on"); // name , value 세팅
  router.replace(`/search?${newSearchParams.toString()}`); // 쿼리 문자열을 작업할 수 있게 replace
};

```

## 3. 인피니트 스크롤링

1. 클라이언트에서 마지막 postId를 서버에 전달하면, 서버에서는 해당 postId 다음부터 post를 보내주기 위해 cursor 를 사용한다.

```ruby

http.get("/api/postRecommends", ({ request }) => {
    const url = new URL(request.url);
    const cursor = parseInt(url.searchParams.get("cursor") as string) || 0; // 다음 post를 위한 cursor
    return HttpResponse.json([
      {
        postId: cursor + 1, // cursor에 1씩 더해서 postId를 sequence화 해준다.
        User: User[0],
        content: `${cursor + 1} Z.com is so marvelous. I'm gonna buy that.`,
        Images: [{ imageId: 1, link: faker.image.urlLoremFlickr() }],
        createdAt: generateDate(),
      },
      {
        postId: cursor + 2,
        User: User[0],
        content: `${cursor + 2} Z.com is so marvelous. I'm gonna buy that.`,
        Images: [
          { imageId: 1, link: faker.image.urlLoremFlickr() },
          { imageId: 2, link: faker.image.urlLoremFlickr() },
        ],
        createdAt: generateDate(),
      },
      {
        postId: cursor + 3,
        User: User[0],
        content: `${cursor + 3} Z.com is so marvelous. I'm gonna buy that.`,
        Images: [],
        createdAt: generateDate(),
      },
      {
        postId: cursor + 4,
        User: User[0],
        content: `${cursor + 4} Z.com is so marvelous. I'm gonna buy that.`,
        Images: [
          { imageId: 1, link: faker.image.urlLoremFlickr() },
          { imageId: 2, link: faker.image.urlLoremFlickr() },
          { imageId: 3, link: faker.image.urlLoremFlickr() },
          { imageId: 4, link: faker.image.urlLoremFlickr() },
        ],
        createdAt: generateDate(),
      },
      {
        postId: cursor + 5,
        User: User[0],
        content: `${cursor + 5} Z.com is so marvelous. I'm gonna buy that.`,
        Images: [
          { imageId: 1, link: faker.image.urlLoremFlickr() },
          { imageId: 2, link: faker.image.urlLoremFlickr() },
          { imageId: 3, link: faker.image.urlLoremFlickr() },
        ],
        createdAt: generateDate(),
      },
    ]);
  }),

```

2. prefetchInfiniteQuery()

- `prefetchInfiniteQuery()` 훅을 사용하여 인피니트 스크롤을 구현
- `initialPageParam` 프로퍼티를 꼭 설정해주어야 하며, 이것이 `cursor` 의 역할

```ruby

export default async function Home() {
  const queryClient = new QueryClient();

  await queryClient.prefetchInfiniteQuery({
    queryKey: ["posts", "recommends"],
    queryFn: getPostRecommends,
    initialPageParam: 0, // pageparam의 초기화
  });

```

3. PostRecommends 컴포넌트 수정

- SSR query에서 보내준 데이터를 받는 PostRecoomends 컴포넌트 또한 `useQuery()` 가 아닌 `useInfiniteQuery()` 로 수정
- useInfiniteQuery를 사용하기 위해서는 그에 맞는 타입을 설정해주어야 하고, `initialPageParam` 과 `getNextPageParam` 을 설정해주어야 한다.

```ruby

"use client";

import { InfiniteData, useInfiniteQuery } from "@tanstack/react-query";
import { getPostRecommends } from "@/app/(afterLogin)/home/_lib/getPostRecommends";
import Post from "@/app/(afterLogin)/_component/Post";
import { Post as IPost } from "@/model/Post";
import { Fragment } from "react";

export default function PostRecommends() {
  const { data } = useInfiniteQuery<
    IPost[],
    Object,
    InfiniteData<IPost[]>,
    [_1: string, _2: string],
    number // initialPageParam의 타입설정
  >({
    queryKey: ["posts", "recommends"],
    queryFn: getPostRecommends,
    initialPageParam: 0,
    getNextPageParam: (lastPage) => lastPage.at(-1)?.postId, // lastPage: 가장 최근에 불러왔던 페이지
    staleTime: 60 * 1000, // fresh -> stale로 변환되는 시간(ms) : 4 WEEK에 나온 개념
    gcTime: 300 * 1000,
  });

  return data?.pages.map((page, idx) => (
    <Fragment key={idx}>
      {page.map((post) => (
        <Post key={post.postId} post={post} />
      ))}
    </Fragment>
  ));
}

```

4. query 함수에 cursor 추가하기

- queryKey를 queryFn에 전달할 수 있었던 것처럼 pageParam 또한 전달한다.

```ruby

type Props = { pageParam?: number };

export async function getPostRecommends({ pageParam }: Props) {
  const res = await fetch(
    `http://localhost:9090/api/postRecommends?cursor=${pageParam}`,
    {
      next: {
        tags: ["posts", "recommends"], // 서버에서 가져온 데이터에 tag를 설정, 이후 캐시 초기화 등에 사용
      },
      cache: "no-store", // next의 tags 를 revalidation (재검증) 하기 전까지는 캐싱이 되어 있다. (1. 의 개념)
    }
  );

  if (!res.ok) {
    throw new Error("Failed to fetch data");
  }

  return res.json();
}

```

## 4. Suspense로 Streaming하여 최적화하기 (feat.loading.tsx, error.tsx)

- page.tsx처럼 next.js에서 제공하는 loading.tsx, error.tsx라는 페이지가 존재한다.
- 해당 페이지가 로딩중일 때는 loading.tsx 페이지가, 로딩이 완료되면 page.tsx, 에러가 발생하면 error.tsx 페이지가 화면에 표시된다.
- 이는 React의 Suspense와 Error Boundary를 활용한 것이다.

### loading.tsx

- 해당 파일은 페이지가 로딩중일때 해당화면을 보여주고 로딩이 끝나면 children에 있는 페이지를 보여준다.

### error.tsx

- 해당 파일은 페이지가 에러가 날 경우 해당화면을 보여준다.

### 콘텐츠 내부에서 로딩 구현하기

- isPending: 초기(데이터를 불러오지 않았을 때)에는 true 
- isFetching: 데이터를 가져오는 순간 true
- isLoading: isPending && isFetching

```ruby

export default function PostRecommends() {
  const { data, fetchNextPage, hasNextPage, isFetching, isPending, isLoading } =
    useInfiniteQuery<
      IPost[],
      Object,
      InfiniteData<IPost[]>,
      [_1: string, _2: string],
      number // initialPageParam의 타입
    >({
      queryKey: ["posts", "recommends"],
      queryFn: getPostRecommends,
      initialPageParam: 0,
      getNextPageParam: (lastPage) => lastPage.at(-1)?.postId, // lastPage: 가장 최근에 불러왔던 페이지들
      staleTime: 60 * 1000, // fresh -> stale로 변환되는 시간(ms)
      gcTime: 300 * 1000,
    });

  // ...

	// isPending인 경우의 로딩창
  if (isPending) {
    <div style={{ display: "flex", justifyContent: "center" }}>
      로딩창
    </div>;
  }

  return data?.map((post) => <Post key={post.postId} post={post} />); // 로딩이 끝난 경우 data가 들어오면 Post 를 보여준다.

```

나머지는 공부중,,,
