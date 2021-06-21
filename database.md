NestJS는 database agnostic입니다. `agnostic` 이라는 것은, 어떠한 플랫폼에 종속되어 있지 않다는 뜻으로, SQL 혹은 NoSQL 데이터베이스에 관계없이 선택적으로 적용할수 있다는 것을 의미합니다.

편의를 위해 NestJS는 `@nestjs/typeorm` 이나 `@nestjs/mongoose` 를 제공하고 있습니다.

## TypeORM Integration

npm i pg

먼저 DB를 준비합니다. **무료**이면서 강력한 기능을 가진 PostgreSQL 데이터베이스와 NestJS를 사용하기 위해 Docker를 이용합니다. 프로젝트의 루트에 다음과 같은 `docker-compose.yml` 을 작성합니다.

```jsx
// docker-compose.yml
version: "3";
services: db: image: postgres;
restart: always;
ports: -"5432:5432";
environment: POSTGRES_PASSWORD: pass123;
```

`docker-compose up -d` 로 데이터베이스의 컨테이너를 띄울 수 있습니다.

npm i @nestjs/typeorm typeorm

TypeORM은 데이터베이스와 NestJS가 효율적이고 안전하게 통신할 수 있게 하는 ORM입니다. 가장 성숙한 ORM이며, 타입스크립트로 작성되어 있습니다.

AppModule에 TypeORM을 **주입**합니다. 주입과 동시에 **초기화(initialize)**하기 위해서 forRoot 메소드와 함께 옵션을 제공합니다.

```jsx
// src/app.module.ts
...
import { TypeOrmModule } from '@nestjs/typeorm';
...
Module({
  imports: [
    CoffeesModule,
    TypeOrmModule.forRoot({
      type: 'postgres', // type of our database
      host: 'localhost', // database host
      port: 5432, // database host
      username: 'postgres', // username(default)
      password: 'pass123', // user password
      database: 'postgres', // name of our database,
      autoLoadEntities: true, // models will be loaded automatically
      synchronize: true, // your entities will be synced with the database(recommended: disable in prod)
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
});
export class AppModule {}/
```

> 배포 시 반드시 `synchronize : false` 를 해야합니다.

PostgreSQL + TypeORM + NestJS 의 설정이 완료되었습니다. 이제 Service 와 Database를 연결할 수 있습니다. `forRoot()` 내부에 설정을 작성하는 것을 선호하지 않는 다면 프로젝트 루트에 `ormconfig.json` 을 생성하고 설정을 작성할 수 있습니다.

이렇게 의존성 주입이 완료되면, TypeORM은 프로젝트의 전체에서 사용될 수 있습니다.

## Repository Pattern

TypeORM은 Repository design pattern을 차용합니다. 각 Entity는 하나의 Repository를 소유하고 있으며, Database 테이블 위에서 이 Repository가 제공됩니다.

`User` 라는 Entity로 예시를 진행합니다.

```jsx
// user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;
}
```

Entity를 작성했습니다. 아직 다른 Module에서는 이 Entity를 인식할 수 없습니다. 따라서 다른 Module에서 이 Entity를 `.forFeature` 메소드를 사용하여 주입합니다.

```jsx
// users.module.ts

import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { UsersService } from "./users.service";
import { UsersController } from "./users.controller";
import { User } from "./user.entity";

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

위 Module은 `forFeature()` 메소드를 현재의 스코프에 해당 Entity의 Repository를 등록하기 위해 사용하였습니다. 이로써 `@InjectRepository()` 를 사용하여 해당 Module의 Service에 Repository를 주입할 수 있습니다.

```jsx
// user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}
...
}
```

## Many to Many relation

Database 내에서 2개 이상의 Table이 서로 연관되어 있을때, Relation을 사용합니다. Many-to-many relation을 정의하기 위해서는 해당되는 데코레이터를 사용합니다. 예를 들어 `Coffee` Entity가 `Flavor` Entity와 Many-to-many relation에 있다고 합니다.

```jsx
// coffees.entity.ts
import {
  Column,
  Entity,
  JoinTable,
  ManyToMany,
  PrimaryGeneratedColumn,
} from 'typeorm';
import { Flavor } from './flavors.entity';

@Entity() // sql table === 'coffee' -> You can change this by providing param in @Entity()
export class Coffee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  brand: string;

	@JoinTable()
	@ManyToMany((type)=>Flavor, (flavor)=>flavor.coffees)
}
```

`@JoinTable()` 은 relation관계에서 주인을 의미합니다. 두 관계 중 한 곳에만 작성하면 됩니다.

`@ManyToMany()` 는 2가지 Parameter를 받습니다. **하나는 relation관계에 있는 Entity. 또 다른 하나는 relation의 반대쪽입니다.** 즉, related entity 내에서 어떤 property가 선택되어야 하는지를 의미합니다.

기본적으로 relation은 response에 포함되지 않습니다. Repository에는 다양한 옵션이 있습니다. 그중에서 relation property를 조작하여, response에 relation을 포함(populate)시킬수 있습니다.

```jsx
// coffees.service.ts
async findone(id: number){
	return this.coffeeRepository.findOne(id, { relations: ['flavors'] })
}
```
