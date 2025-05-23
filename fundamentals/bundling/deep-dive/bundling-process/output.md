# 출력

출력(Output)은 웹팩이 번들링한 후 최종적으로 생성하는 번들 파일이나 출력 디렉토리를 의미해요.
번들 파일은 브라우저에서 실행할 수 있도록 변환된 JavaScript 코드로, 여러 개의 모듈이 하나의 파일로 묶이거나 여러 개의 청크 파일로 이루어져 있어요.

웹팩 설정 파일의 `output` 필드에서 번들 파일이 저장될 경로, 파일명, 출력 형식 등을 설정할 수 있어요. 이를 활용하면 코드 스플리팅과 캐시 무효화를 적용해 자원을 효율적으로 로드할 수 있어요.

이제 자주 사용하는 옵션들을 하나씩 살펴볼게요.

## 번들 파일 이름 설정하기:

`filename`은 생성된 번들 파일의 이름을 설정하는 옵션이에요.

### 1. 단일 엔트리 구문에서 설정하기

[단일 엔트리 구문](../bundling-process/entry#단일-엔트리-구문)에서 번들 파일의 이름은 다음과 같이 설정할 수 있어요.

:::tabs key:bundler-single-entry

=== Webpack

```tsx
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
};
```

=== Vite

```ts
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      input: './src/index.tsx',
      output: {
        entryFileNames: 'bundle.js',
      },
    },
  },
});
```
:::

`yarn build` 명령을 실행하면, `filename`으로 설정했던 이름으로 번들 파일이 출력된 것을 확인할 수 있어요.

```text{2}
├─ dist
│   └─ bundle.js
```

### 2. 객체 구문에서 설정하기

[객체 구문](../bundling-process/entry#객체-구문)에서 플레이스홀더를 사용하면, 각 번들 파일의 이름이 `entry` 필드의 `key`에 맞춰 자동으로 할당돼요. 즉, 번들 파일의 이름을 동적으로 설정할 수 있어요.

예를 들어, 다음과 같이 `filename`에 플레이스홀더 `[name]`을 적용해볼게요.


:::tabs key:bundler-object-entry

=== Webpack

```tsx
// webpack.config.js
const path = require('path');

module.exports = {
  entry: {
    app: './src/index.tsx',
    adminApp: './src/admin.ts',
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

=== Vite

```ts
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        app: './src/index.tsx',
        adminApp: './src/admin.ts',
      },
      output: {
        entryFileNames: '[name].bundle.js',
      },
    },
  },
});
```

:::


`yarn build` 명령을 실행하면, `entry` 필드의 key인 `app`와 `adminApp`이 각각 번들 파일의 이름에 반영돼요.

```text{2-3}
├─ dist
│  ├─ app.bundle.js
│  └─ adminApp.bundle.js
```

::: info 플레이스홀더(placeholder) 이해하기

플레이스홀더는 동적으로 값을 할당하기 위해 사용하는 형식이에요. 코드나 설정 파일에서 특정 위치에 지정해 두면, 실행 시점에 해당 값이 자동으로 대체돼요.

그래서 파일 이름을 설정할 때 플레이스홀더를 사용하면, 번들링 과정에서 각각의 엔트리 포인트의 이름이나 파일 해시값으로 대체돼요.

- `[name]`, `[id]`: 엔트리 포인트의 이름이나 청크 파일의 ID를 파일 이름에 포함할 때 사용해요.
- `[hash]`: 전체 번들링 결과에 대한 해시값이에요. 빌드 전체가 변경될 때마다 새로운 해시값이 생성돼요.
- `[contenthash]`: 파일의 내용이 변경됐을 때만 새로운 해시값이 생성돼요. 이 값을 활용하면 변경된 파일만 캐시가 무효화돼요.
- `[chunkhash]`: 청크 파일 단위로 생성되는 해시값이에요. 같은 청크 파일 내에서는 같은 해시값을 공유하고, 청크 파일이 변경될 때만 해시값이 갱신돼요.

:::

## 빌드 전 출력 디렉토리를 초기화하기:

빌드할 때마다 이전 결과물을 자동으로 삭제할 수 있어요.  
이렇게 하면 출력 디렉토리에 불필요한 파일이 쌓이는 것을 방지하고, 항상 최신 번들 파일만 유지할 수 있어요.

`clean` 옵션을 `true`로 설정하면, 새로운 번들 파일을 생성하기 전에 기존의 출력 디렉토리를 정리해요.

:::tabs key:bundler-clean-option

=== Webpack

```tsx
// webpack.config.js
module.exports = {
  output: {
    clean: true,
  },
};
```

=== Vite
Vite는 clean 옵션이 기본 적용되어 별도 설정이 필요 없어요.

:::



## 파일 이름을 동적으로 설정하기

`filename`과 `chunkFilename`옵션에서 번들 파일과 청크 파일의 이름을 동적으로 지정할 수 있어요. 이렇게 하면 코드 스플리팅, 캐시 무효화 등의 최적화 전략을 쉽게 적용할 수 있어요.

### 코드 스플리팅

코드 스플리팅으로 분리하려는 파일 이름에 `[id]`나 `[name]` 같은 플레이스홀더를 사용해 파일 이름을 설정할 수 있어요.


:::tabs key:bundler-splitting-hash

=== Webpack

```tsx
// webpack.config.js
module.exports = {
  output: {
    filename: "[name].bundle.js",
    chunkFilename: "[id].js",
  },
};
```

=== Vite

```ts
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        entryFileNames: "[name].bundle.js",
        chunkFileNames: "[name].bundle.js",
      },
    },
  },
});
```
:::


또한 이름은 함수로 작성할 수도 있는데, 이 함수는 **파일 경로와 관련된 메타데이터 객체**를 인자로 받아요. 이를 활용하면 각 파일의 이름이나 타입에 따라 동적으로 파일 이름을 설정할 수 있어요.

### 해시를 활용한 캐시 무효화

브라우저는 청크 파일의 이름이 동일하면 기존에 캐시된 파일을 계속 사용하기 때문에, 변경 사항이 있어도 새로운 청크 파일을 가져오지 못해요. 이로 인해 UI가 최신 상태로 표시되지 않거나, 기능이 업데이트되지 않는 문제가 발생할 수 있어요.

이 문제를 방지하려면 번들 파일에는 `[contenthash]`, 청크 파일에는 `[chunkhash]` 플레이스홀더를 사용해서 파일마다 고유한 해시 값을 할당하면 돼요.

이렇게 하면 파일 내용이 변경될 때마다 해시 값이 갱신돼서, 브라우저가 이를 감지해 새로운 파일을 로드할 수 있어요. 덕분에 캐시 무효화가 효과적으로 이루어져 항상 최신 파일을 제공할 수 있고, 변경된 파일만 다시 로드되므로 불필요한 리소스 로드를 줄일 수 있어요.

:::tabs key:bundler-splitting-hash

=== Webpack

```tsx
// webpack.config.js
module.exports = {
  output: {
    filename: '[name].[hash].js',
    chunkFilename:'[name].[hash].js',
  },
};
```

=== Vite

```ts
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        entryFileNames: '[name].[hash].js',
        chunkFileNames: '[name].[hash].js',
      },
    },
  },
});
```
:::

반면, 규모가 작거나 변경이 자주 일어나지 않는 프로젝트라면 번들 파일 자체의 고유한 `[hash]` 하나로 단순하게 관리할 수도 있어요. 이 경우 빌드할 때마다 동일한 해시가 적용되어 캐시 활용도는 줄어들지만 설정이 간단해지는 장점이 있어요.

:::info 청크 파일 이해하기

청크(chunk)는 웹팩이 애플리케이션 실행에 필요한 모듈을 묶어 만든 코드의 단위예요.

웹팩은 애플리케이션이 브라우저에서 실행될 때 필요한 파일을 청크 파일로 나눠 관리해요. 이렇게 분리된 청크 파일은 브라우저가 필요한 시점에 동적으로 로드할 수 있어 성능을 최적화하는 데 도움이 돼요.

웹팩이 청크 파일을 관리하는 방식에 따라 브라우저에서 파일을 로드하는 방식도 달라져요.

**1. 모든 청크 파일을 하나의 번들 파일로 묶어 로드하기**: 초기 로딩 속도가 느려질 수 있지만 이후 추가 요청 없이 실행할 수 있어, 트래픽이 적고 규모가 작은 프로젝트에 적합해요.

**2. 각 청크 파일을 독립된 파일로 분리해 로드하기**: 초기 로딩 속도가 빨라지고, 코드 변경이 있을 때 필요한 부분만 갱신할 수 있어 대규모 애플리케이션에 유용해요.

개발자 도구의 **네트워크 탭**에서 확인해 보면, 여러 개의 JavaScript 파일이 로드되는 것을 볼 수 있어요.

:::