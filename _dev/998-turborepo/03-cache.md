---
last_update:
  date: 2023-02-25
---

# 모노레포 캐싱

[코어 컨셉 - 캐싱](https://turbo.build/repo/docs/core-concepts/caching) 요약

우린 package.json에 정의한 스크립트(build, test, lint 등)를 실행한다. 터보레포에선 이를 태스크라 부름.

터보레포는 태스크의 결과물이나 로그를 캐싱할 수 있다. 이는 엄청난 속도 향상으로 이어진다.

### 캐시가 없을 때

각 태스크는 인풋과 아웃풋을 가진다. 예를 들어 빌드명령어는 인풋으로 소스파일을 가지고 아웃풋으로 빌드 결과물고 로그를 내보낸다.

1. 터보레포는 먼저 태스크의 인풋을 평가하고 (기본적으로 워크스페이스 폴더 내 git이 추적하는 모든 파일) 해싱값을 붙인다. (예 - 78awdk123)
2. 로컬 파일 시스템에서 캐시 파일이 있는지 확인한다. (./node_modules/.cache/turbo/78awdk123)
3. 만약 터보레포가 캐싱값을 찾지 못하면 태스크를 실행한다.
4. 태스크를 완료한 후 터보레포는 모든 파일과 로그를 캐싱하여 저장한다.

### 캐시가 있을 때

1. 동일한 인풋에서는 캐싱의 해시값이 변하지 않는다.
2. 로컬 파일 시스템에서 캐시파일이 있는지 확인하고, 있다면 태스크를 실행하는 대신 캐싱값으로 대신한다.

### 출력 설정

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "outputs": [".next/**", "!.next/cache/**"],
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

빌드의 결과물에 해당되는 경로를 glob패턴으로 지정한다. 만약 lint와 같이 출력 파일이 없는 경우 Output을 생략하면 결과 로그만 캐싱함.

### 입력 설정

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    // ... omitted for brevity

    "test": {
      // A workspace's `test` task depends on that workspace's
      // own `build` task being completed first.
      "dependsOn": ["build"],
      // A workspace's `test` task should only be rerun when
      // either a `.tsx` or `.ts` file has changed.
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"]
    }
  }
}
```

기본적으로 워크스페이스 내에서 깃이 추적하는 파일 중 어느 파일이든 변경이 있으면 작업 공간이 업데이트된 것으로 간주된다. 그러나 특정 파일이 변경되었을 때만 업데이트가 필요한 태스크가 있을 수 있다. 이러한 경우 inputs 배열에 업데이트를 판별할 조건을 지정할 수 있다.

### 캐싱 제외

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### 환경 변수 값에 따른 캐싱 변경

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      // env vars will impact hashes of all "build" tasks
      "env": ["SOME_ENV_VAR"],
      "outputs": ["dist/**"]
    },

    // override settings for the "build" task for the "web" app
    "web#build": {
      "dependsOn": ["^build"],
      "env": [
        // env vars that will impact the hash of "build" task for only "web" app
        "STRIPE_SECRET_KEY",
        "NEXT_PUBLIC_STRIPE_PUBLIC_KEY",
        "NEXT_PUBLIC_ANALYTICS_ID"
      ],
      "outputs": [".next/**"]
    }
  }
}
```

업데이트 여부를 판별할 때 환경변수값도 명시해줄 수 있다.

신기한건 NEXT*PUBLICK*\* 와 같은 프레임워크에서 정해져 잇는 공개 환경변수는 터보에서 자동으로 추론하고 포함하기 때문에 생략 가능하다.
