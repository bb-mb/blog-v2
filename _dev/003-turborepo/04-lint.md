---
last_update:
  date: 2023-02-25
---

공식 docs 참고 ([tsconfig](https://turbo.build/repo/docs/handbook/linting/typescript), [eslint](https://turbo.build/repo/docs/handbook/linting/eslint))

## tsconfig 설정

packages/tsconfig 디렉토리에 패키지를 설치했다.

```json
// package.json
{
  "name": "@packages/tsconfig",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "private": true
}

// nest.json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "es2017",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": false,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "forceConsistentCasingInFileNames": false,
    "noFallthroughCasesInSwitch": false
  }
}
```

nestjs 설치 때 자동으로 설정된 tsconfig를 그대로 옮겼다.

```json
// apps/server/tsconfig
{
  "extends": "@packages/tsconfig/nest.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

`yarn workspace server add @packages/tsconfig@* -D` 스크립트로 서버 workspace 내에 tsconfig 패키지를 설치한다.

extends에 tsconfig 설정파일을 지정하면 적용된다.

alias 설정은 워크스페이스별로 다를 것이라 별도로 추가설정함.

## eslint 설정

tsconfig 와 동일하다.

@pakages/eslint-config-custom 이름으로 패키지를 생성했다.

```json
// package.json
{
  "name": "@packages/eslint-config-custom",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "private": true,
  "dependencies": {
    "eslint-config-prettier": "^8.6.0"
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^5.53.0"
  }
}
```

```jsx
// index.js
module.exports = {
  plugins: ['@typescript-eslint/eslint-plugin'],
  extends: ['prettier'],
  rules: {
    '@typescript-eslint/no-explicit-any': 'error',
  },
};
```

위와 같이 패키지를 만들고 공통으로 적용할 린트 설정을 만든다.

```json
// apps/web/.eslintrc.json
{
  "extends": ["next/core-web-vitals", "@packages/eslint-config-custom"]
}
```

tsconfig와 마찬가지로 앱 워크스페이스에서 패키지를 설치하고 extends에 적용해주자.

추가로 궁금해서 해봤는데 extends에 설정된 eslint 설정 내용이 충돌하면 어떻게 될까?

```json
// apps/web/.eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "@packages/eslint-config-custom",
    "@packages/eslint-config-custom2"
  ]
}
```

위 커스텀 eslint 설정과 동일하고 '@typescript-eslint/no-explicit-any'룰을 off로 준 custom2 패키지를 만들고 적용해 봄. 확인해보니 해당 설정이 off로 동작한다. extends의 더 뒤에 설정된 룰이 앞의 룰을 덮어쓰는듯. 너무 당연한가?
