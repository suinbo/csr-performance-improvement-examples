# CSR 렌더링 및 로딩 성능 개선 예제

### Environment

-   [nodejs] - v.18.18.2
-   [reactjs] - v.18.2.37
-   [vite] - v.5.0.7

### Main Plugins

-   [react-router-dom] - Routing (v.6.20.1)
-   [react-query] - Data caching & Auto fetch (v.3.39.3)

### Development

will be run at local-environment

```bash
yarn dev
```

### Setting

-   Node.js 모듈 사용시

    -   tsconfig.json에 아래 설정 추가

    ```
    {
      "compilerOptions": {
        "types": ["node"]
      }
    }
    ```

    -   @types/node 패키지 설치: `npm install --save-dev @types/node`

### Note

```
웹에서 렌더링 및 로딩 시 발생하는 성능 저하 문제에 대해 분석하고 개선 방법을 찾아본다.
```

1.  react-query를 할용하여 하나의 컴포넌트에서 여러 개의 데이터 호출하는 경우

    -   **[문제]** 선언적 로딩 처리를 위해 두 개의 useQuery에 `suspense: true` 옵션을 추가하면 API 가 순차적으로 호출 되는데,  
        호출하는 API 수가 늘어날수록 화면 렌더링 끝나는 시점이 느려짐 (비효율)

        -   **[분석 1]**  
            두 개의 useQuery에 `suspense: true` 옵션을 추가하면 (선언적 로딩 처리 시),  
            첫번째 쿼리가 내부적으로 Promise 를 발생시키고 다른 쿼리가 실행되기 전에 컴포넌트를 일시 중단하여
            순차적으로 API 호출이 발생

            ![![Alt text](image.png)](src/assets/image1.png)

        -   **[분석 2]**  
            두 개의 useQuery에 `suspense: true` 옵션을 제거하면 병렬적 API 호출이 가능하나,  
            API 호출 시 로딩처리에 대한 부가적인 로직이 필요함. (선언적 로딩 처리 불가)

            ```
            const { data: userData, isLoading: isUserLoading } = useQuery({...})
            const { data: bannerData, isLoading: isBannerLoading } = useQuery({...})

            if (isUserLoading || isBannerLoading) {
              return <Loading />
            }
            ```

    -   **[성능 개선]**

        -   **[방법 1]**  
            useQueries 훅을 이용하여 useQuery 훅에 전달하던 매개변수들을 배열형태로 전달
            => 병렬 API 요청

            ![![Alt text](image.png)](src/assets/image2.png)

        -   **[방법 2]**
