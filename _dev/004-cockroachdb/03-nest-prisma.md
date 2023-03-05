# nest prisma 연결

[깃헙 링크](https://github.com/soso01/cockroachdb-test)

```tsx
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "cockroachdb"
  url      = env("DATABASE_URL")
}

model Post {
  id        BigInt   @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  title     String   @db.Char(255)
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  BigInt
}

model Profile {
  id     BigInt  @id @default(autoincrement())
  bio    String?
  user   User    @relation(fields: [userId], references: [id])
  userId BigInt  @unique
}

model User {
  id      BigInt   @id @default(autoincrement())
  email   String   @unique
  name    String?
  posts   Post[]
  profile Profile?
}
```

prisma의 스키마 파일을 위와 같이 정의함

```tsx
import { INestApplication, Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }

  async enableShutdownHooks(app: INestApplication) {
    this.$on('beforeExit', async () => {
      await app.close();
    });
  }
}
```

PrismaService를 정의하고

```tsx
import { PrismaService } from '@/prisma/prisma.service';
import { faker } from '@faker-js/faker';
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  constructor(private prisma: PrismaService) {}

  findUserById = async (id: bigint) => {
    return await this.prisma.user.findUnique({ where: { id } });
  };

  findUserMany = () => {
    return this.prisma.user.findMany();
  };

  createUser = () => {
    return this.prisma.user.create({
      data: {
        name: faker.name.fullName(),
        email: faker.internet.email(),
      },
    });
  };
}
```

사용할 도메인 서비스에서 주입받아 사용.
prisma client에서 타입을 자동으로 생성했는지 자동완성이 잘 된다.

주의할 점은 bigint관련해서 신경써줘야 할 부분이 있다.
find where문에서도 id값을 bigint로 입력하지 않으면 오류가 발생하여 따로 타입 변환이 필요함.
