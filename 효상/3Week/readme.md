# 3Week

## 1. usePathname과 /explore 페이지

### /explore 페이지

- /explore 페이지는 레이아웃에 있던 나를 위한 트렌드, 검색 부분이 우측 섹션에서 가운데 섹션으로 이동된다.

### usePathname() 훅

- 현재 URL의 pathname 문자열을 반환한다.

```ruby
// TrendSection.tsx

import style from "./trendSection.module.css";
import Trend from "./Trend";
import { usePathname } from "next/navigation";

export default function TrendSection() {
  const pathname = usePathname();

  if (pathname === "/explore") return null;

  return (
    <div className={style.trendBg}>
      <div className={style.trend}>
        <h3>나를 위한 트렌드</h3>
        <Trend />
        <Trend />
        <Trend />
        <Trend />
        <Trend />
        <Trend />
        <Trend />
        <Trend />
        <Trend />
        <Trend />
      </div>
    </div>
  );
}
```

## 2. useSearchParams와 프로필, /search 페이지

### Tab 컴포넌트

- 홈에서 사용되는 Tab 컴포넌트와 탐색하기 페이지에서 사용되는 Tab 컴포넌트는 역할이 다름 
- 홈에는 Tab을 활성화하면 주소창에 변화가 없고 탐색하기 페이지의 Tab은 활성화 될 때마다 주소창에 변화가 생긴다
- 이럴 때에는 각각의 컴포넌트로 만들어주는 것이 좋다.

### useRouter()

- router 객체에 접근할 수 있는 훅

```ruby
// router 객체에 접근
const router = useRouter();
const searchParams = useSearchParams();

// click 마다 replace로 전환해준다.
const onClickHot = () => {
  setCurrent("hot");
  router.replace(`/search?q=${searchParams.get("q")}`);
};
```
## 3. 이벤트 캡처링과 /status/[id] 페이지

### Link와 useRouter의 차이

- Link : hover 했을 때 브라우저 좌측 하단에 이동할 주소가 보여짐
- useRouter : useRouter를 통하여 onClick 동작으로 이동될 경우 좌측 하단에 이동할 주소가 보여지지 않는다.

* 주의 : onClick 이벤트로 이동할 경우 SEO에 유리하지 않을 수 있음 (SE가 click이벤트 추적 불가능)

- Post 컴포넌트에서의 클릭 이벤트는 PostArticle과 children인 Link 들이 있다.
- 이때 PostArticle 내의 children인 Link를 클릭했을 때에도 PostArticle 클릭 이벤트가 동작되는 현상이 발생
- 이러한 현상을 이벤트 캡쳐링 이라 하며 이때는 onClick 핸들러 외에 onClickCapture 핸들러를 사용한다
- 캡쳐링 단계에 PostArticle의 클릭 이벤트가 동작하도록 변경해서 이러한 현상을 해결할 수 있다.

* PostArticle

```ruby
export default function Post() {

  return (
    <PostArticle post={target}>
      <div className={style.postWrapper}>
        <div className={style.postUserSection}>
          <Link href={`/${target.User.id}`} className={style.postUserImage}>
            <img src={target.User.image} alt={target.User.nickname} />
            <div className={style.postShade} />
          </Link>
        </div>
        <div className={style.postBody}>
          <div className={style.postMeta}>
            <Link href={`/${target.User.id}`}>
              <span className={style.postUserName}>{target.User.nickname}</span>
              &nbsp;
              <span className={style.postUserId}>@{target.User.id}</span>
              &nbsp; · &nbsp;
            </Link>
            <span className={style.postDate}>
              {dayjs(target.createdAt).fromNow(true)}
            </span>
          </div>
          <div>{target.content}</div>
          <div className={style.postImageSection}></div>
          <ActionButtons />
        </div>
      </div>
    </PostArticle>
  );
}
```

* onClick 핸들러 외에 onClickCapture 핸들러를 사용하여 PostArticle의 클릭 이벤트가 동작하도록 변경

```ruby
export default function PostArticle({ children, post }: Props) {
  const router = useRouter();

  const onClick = () => {
    router.push(`/${post.User.id}/status/${post.postId}`);
  };

  return (
    <article onClickCapture={onClick} className={style.post}>
      {children}
    </article>
  );
}
```

### 상세 이미지 페이지 만들기

- 이미지 모달 형태임으로 패러렐 라우트와 인터셉팅 라우트로 만들어준다. (홈 컴포넌트 내부)

```ruby
import Home from "@/app/(afterLogin)/home/page";

type Props = {
  params: { username: string; id: string; photoId: string };
};

// spring이란 비슷한 듯
export default function Page({ params }: Props) {
  params.username; // elonmusk
  params.id; // 1
  params.photoId; // 1

  return <Home />;
}
```

## 4. 서버 컴포넌트에서 Server Actions 사용하기

- Server Actions 함수 작성

```ruby
const submit = async (formData: FormData) => {
    "use server";

    // id password 검증
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

    // shouldRedirect : home으로 redirect
    let shouldRedirect = false;

    try {
      const response = await fetch(
        `${process.env.NEXT_PUBLIC_BASE_URL}/api/users`,
        {
          method: "post",
          body: formData,
          credentials: "include", // 쿠키 전달 속성
        }
      );

			// 중복 사용자 체크
      if (response.status === 403) {
        return { message: "user_exists" };
      }

      shouldRedirect = true;
    } catch (err) {
      console.error(err);
      shouldRedirect = false;
    }

    if (shouldRedirect) {
      redirect("/home");
    }
  };
};
```