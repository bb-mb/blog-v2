---
last_update:
  date: 2023-03-05
---

# Prisma ‘Do not know how to serialize a BigInt’ 이슈

[https://github.com/prisma/studio/issues/614](https://github.com/prisma/studio/issues/614)

CockroachDB는 id값으로 BigInt 타입을 쓴다.

BigInt 타입을 toJSON 메서드로 직렬화하는데 문제가 있는 듯.

```tsx
(BigInt.prototype as any).toJSON = function () {
  return this.toString();
};
```

앱 루트에 다음 코드를 추가하면 해결할 수 있다.
