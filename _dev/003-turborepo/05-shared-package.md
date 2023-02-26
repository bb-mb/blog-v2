---
last_update:
  date: 2023-02-26
---

# 내부 패키지 코드 공유

[https://turbo.build/repo/docs/handbook/sharing-code/internal-packages](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages) 참고

```tsx
// index.ts
export function add(a: number, b: number) {
  return a + b;
}
```

@packages/my-lib 이라는 내부 패키지를 생성하고 간단한 add 함수를 추가했다.

이 함수를 web, server에 import해서 사용해보자.

## web - nextjs

외부 패키지를 import 하기 위해서 트랜스파일링이 필요하다. next는 13버전부터 next.config.js 에서 설정가능하다.

```tsx
// next.config.js

/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  transpilePackages: ['@packages/*'],
};

module.exports = nextConfig;
```

@packages/\* 패턴으로 설정 가능해서 이렇게 했다. @packages 를 내부 패키지의 네이밍 규칙으로 잡으면 추가 설정 변경이 없어 편할 것 같다.

```tsx
import { add } from '@packages/my-lib';

export default function Home() {
  return (
    <main>
      <p>20 + 10 : {add(20, 10)}</p>
    </main>
  );
}
```

일반 패키지 사용하듯 import 해서 쓰면 된다.

![01.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da9ef6cb-8a26-44a8-b45f-39a3b53ee503/01.png)

작동 확인

## server - nestjs

nextjs와 다르게 nest에는 트랜스파일 설정 옵션이 없다.

직접 import하면 ts 관련 오류가 생기는데 찾아본 [해결방안](https://stackoverflow.com/questions/73189881/turborepo-package-unexpected-token-export)으로는 패키지 내에서 트랜스파일 후 번들파일을 참조하는 것이다.

my-lib을 수정하자

```tsx
// package.json
{
  "name": "@packages/my-lib",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "license": "MIT",
  "private": true,
  "dependencies": {
    "@packages/tsconfig": "*"
  },
  "scripts": {
    "build": "tsc"
  }
}

// tsconfig
{
  "extends": "@packages/tsconfig/nest.json",
  "compilerOptions": {
    "outDir": "dist"
  }
}

```

package.json에서 메인파일을 트랜스파일 된 ./dist/index.js 파일로 설정한다. 빌드 스크립트를 추가하고, tsconfig는 outDir로 dist 폴더를 설정한다.

```tsx
// turbo.json - root
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "outputs": [".next/**", "dist/**"],
      "dependsOn": ["^build"]
    },
    "lint": {},
    "dev": {
      "persistent": true,
      "cache": false,
      "dependsOn": ["^build"]
    }
  }
}
```

루트의 turbo.json 수정도 필요하다. server의 빌드나 dev 스크립트를 실행하기 전에 의존 패키지(my-lib)의 빌드가 필요하다. 의존 패키지의 빌드 명령어 후 스크립트를 실행하도록 dependsOn: “^build” 를 추가한다.

```tsx
import { Injectable } from '@nestjs/common';
import { add } from '@packages/my-lib';

@Injectable()
export class AppService {
  getHello(): string {
    return `Hello World! - (${add(10, 20)})`;
  }
}
```

nestjs의 기본 app.service.ts 파일에서 함수를 가져와서 사용했다. 잘 된다.

그런데 이렇게 트랜스파일을 미리 하니 Nextjs 에서 위의 transpilePackage 옵션이 없어도 잘 작동한다. 저 설정을 지워야할지 고민된다.

---

추가) tsc대신 [tsup](https://tsup.egoist.dev/)을 사용하면 파일을 하나로 묶을 수 있음
