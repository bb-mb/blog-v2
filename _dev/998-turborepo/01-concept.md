---
last_update:
  date: 2023-02-21
---

# 터보레포 코어 컨셉

### 모노레포

모노레포는 많은 장점이 있지만 확장이 어렵다. 각 작업 공간에서의 스크립트를 합쳐서 단일 리포지토리에 실행할 태스크가 수백개가 있을 수 있음.

터보레포는 모노레포의 스케일링 문제를 해결한다.

### 모노레포에서 작업 실행

터보레포는 멀티 태스킹이 가능하다. 각각의 모든 앱이 lint 작업을 끝마친 후 test 작업을 수행하고 모두 마치면 build하는 동기적인 동작을 터보레포는 병렬적으로 처리 가능함.

파이프라인은 모노레포에서 서로 의존하는 태스크를 선언한다.

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", ".svelte-kit/**"]
    },
    "deploy": {
      // A workspace's `deploy` task depends on the `build`,
      // task of the same workspace being completed.
      "dependsOn": ["build"]
    }
  }
}
```

위 설정에서 build는 depondsOn : “^build” 의존성을 가지는데 이는 의존하는 워크스페이스의 build작업이 완료된 후 실행됨을 의미한다. (web이라는 app에서 modal이라는 패키지를 의존한다면 modal 패키지의 빌드작업이 완료된 후 web의 build 작업이 시작된다.)

deploy는 build작업이 완료된 후 실행된다.

사용자가 직접 의존성을 지정해줄 수도 있다. (프론트 앱에서 의존성이 전혀 없는 백엔드 빌드가 끝난 후 실행하도록 하는 등)

### 워크스페이스 필터링

**`turbo run build --filter='./apps/*'`**

위와 같은 명령어로 특정 워크스페이스만 필터링할 수 있다.

### 작업 공간 구성

turbo.json 설정 파일은 루트에서 전역으로 설정된다. 각 앱에서 개별 turbo.json 파일을 생성해 전역 설정을 확장하거나 덮어쓸 수 있다.
