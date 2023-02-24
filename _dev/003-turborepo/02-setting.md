---
last_update:
  date: 2023-02-24
---

## 단일 레포

단일 레포지토리 설정에서 애플리케이션 간에 코드를 공유하려면?

app, docs 두 앱이 있고, shared-utils라는 npm에 패키지로 배포된 모듈에 의존한다고 하자.
shared-utils에 심각한 버그가 생겼을 때 아래와 같이 대처해야 한다.

1. shared-utils 오류 수정 및 커밋
2. npm에 배포
3. app 에서 shared-utils의 새 버전으로 커밋
4. docs 에서 shared-utils의 새 버전으로 커밋
5. app과 docs 배포

이 과정은 오래걸린다.

## 모노레포

단일 레포 설정에서는 2~4단계의 과정이 생략되어도 된다. npm의 버전에 의존하지 않기 때문에 버전관리도 필요하지 않다.

이를 통해 여러 앱과 패키지의 버그를 한 번에 수정하는 단일 커밋을 생성하고 팀의 속도를 크게 향상시킬 수 있다.

## workspace

workspace는 모노 레포의 빌딩 블록이다. 모노 레포에 추가하는 각 앱과 패키지는 자체 작업 공간에 있다.

패키지 매니저 npm, yarn, pnpm중에 선택하자. pnpm이 가장 빠르다고는 하는데 나는 대중적인 yarn을 사용했다.

```json
// package.json
{
  "name": "turborepo",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "workspaces": ["apps/*", "packages/*"]
}
```

최상단 루트에 apps, packages 디렉토리를 만들고 workspace영역으로 선언해준다.

apps 디렉토리에 상용 애플리케이션을 설치한다. 현재 web(next.js프레임워크) 와 server(nestjs) 설치함.

**`yarn workspace <workspace> <script` 명령어로 각 워크스페이스에 정의된 스크립트를 실행할 수 있다.**

## 터보레포 설정

**`yarn global add turbo`**

글로벌로 turbo 설치

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "outputs": [".next/**", "dist/**"]
    },
    "lint": {}
  }
}
```

프로젝트 루트에 turbo.json을 추가한다.

위의 설정으로 turbo build 를 실행하면 미리 설치해둔 web, server 디렉토리의 build 스크립트가 실행된다.

outputs는 캐싱할 빌드의 결과물을 지정한다. 1회 빌드 스크립트 실행 후 outputs에 해당하는 디렉토리를 삭제했는데 다시 build 스크립트를 실행하니 빌드가 즉시 완료되고 삭제한 빌드 결과물이 생김. 신기했다. (찾아보니까 node_modules/.cache/turbo 에 저장된다)
