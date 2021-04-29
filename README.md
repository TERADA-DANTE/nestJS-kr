# main.ts

이름을 변경할 수 없다.

`AppModule` 로 앱을 실행시킨다.

# AppModule

> nest generate module

Module : 애플리케이션의 일부분, Django에서의 App

main.ts 가 참조하는 Root module.

AppModule은 AppController와 AppService 이외를 가질 수 없다. 즉 회원 등록, 열람, 수정, 삭제기능을 하는 User 모듈이 존재한다고 하면 다음과 같이 작성된다.

```jsx
main.ts < -AppModule;

// app.module.ts
@Module({
  imports: [UserModule],
  controllers: [AppController],
  provider: [AppService],
})
// users.module.ts
@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

provider에 Service를 추가함으로써, Controller에서 Service의 import없이 타입 추가만으로 잘 작동하는 것을 알 수 있다. 이것을 Dependency Injection이라고 한다. Service에 있는 데코레이터 `@Injectable` 가 이것을 의미한다.

# Controller

> nest generate controller

express에서 router같은 존재로, url을 가져오고 함수를 실행한다

Ex) @Get 데코레이터는 express의 get 라우터와 같은 역할

```jsx
@Get('/hello')
sayHelloWorld() : string {
	return 'Hello world'
}

// localhost:3000/hello -> Hello world
```

데코레이터는 데코레이터의 함수나 클래스와 붙어있어야 한다. 개행이 들어가면 작동할 수 없다.

이외에도 `Post` , `Delete`, `Put` , `Patch` 등도 같은 방식으로 작성한다. 파라미터를 받는 방법에 대해서는 아래에서 설명한다.

PUT 메소드는 모든 리소스를 업데이트하기 때문에 일부 리소스를 업데이트하는 PATCH 메소드를 사용하는 것이 바람직하다.

```jsx
@Controller('/users')
export class UserController{
	// constructor는 아래 Service에서 설명
	constructor(private readonly userService: UserService){}

	@Get()
	getUsers(){
		return 'All users profile'
	}

	@Get('/:id')
	getUserById(@Param('id') id: number){
		return 'A user profile specified by ID'
	}

	@Patch('/:id')
	updateUserById(@Param('id') id: number, @Body() userData){
		return 'A user profile specified by ID which has updated'
	}
}
```

위의 경우 `@Get()` 가 `'/'` 을 라우팅한다. 하지만 `@Controller('/users')` 는 `'/users'` 스코프를 의미하기 때문에(엔트리 포인트) 결국 `[localhost:3000/users](http://localhost:3000/foo)` 로 라우팅이 생성된다.

`@Param('id') id: string` 에서 `@Param('id')` 는 url에서 id가 파라미터임을 지시해준다. 이후 받아온 파라미터를 이하 로직에서 id라는 변수(`string`)에 담아사용한다. 즉 변수 이름은 변경할 수 있다.

즉,`'/:id'` 는 그 자체로는 로직안에서 파라미터로 작용할 수 없다. 오로지 다이나믹 라우팅 만을 담당한다.

Restful API 를 작성하기 위해서는 Controller의 이름을 복수형으로 할 필요가 있다.

request 의 body를 받는 경우에는 `@Body() reqBody : type` 의 형식으로 작성한다. NestJS에서는 JSON을 string으로 변환하거나 역변환하는 설정이 필요없다.

`/serach?name=uniqueName` 과 같은 방식의 get params의 라우팅은 다음과 같다.

```jsx
@Get('/search')
search(@Query("name"), userName : string){
	return 'A user'
}
```

위의 코드가 `Get('/:id')` 보다 아래에 있다면 이것은 `'/:id'` 로 인식되어 작동하지 않는다. Get(Patch)과 Post가 각각 `@Query(parameter)` , `@Body()` 라는 것에 주의한다.

## Express

NestJS는 Express위에서 동작하고 있다. 즉, 필요에 따라 express의 request, response에 접근할 수 있다.

```jsx
@Get()
GetUserById(@Req() request, @Res() response){}
```

그럼에도 불구하고 위와 같은 방식은 추천되지 않는다.

이유는 NestJS가 fastify와 호환되는 기능을 가지고 있기 때문이다. NestJS는 express와 fastify 두 개의 Framework 위에서 동작하기 때문에, Express의 객체에 직접적으로 접근하는 것은 미래를 생각하지 않는 개발 습관이다.

## DTO

데이터 모델은 Entity(아래에 있음) 에서 관리한다. 그런데, Patch나 Create 메소드의 경우 body로 들어오는 데이터의 모델은 어떻게 할까?

DTO는 데이터 전송 객체(Data Transfer Object)로, 다음과 같다.

```jsx
// /dto/create-user.dto.ts
export class CreateUserDto{
	readonly name : string
	readonly age : number
	readonly alias : string[]
}

// /dto/update-user.dto.ts
export class UpdateUserDto{
	readonly name? : string
	readonly age? : number
	readonly alias? : string[]
}

```

이렇게 만들어진 DTO는 Controller와 Service에서 type interface처럼 사용할 수 있다. 즉, 개발 경험(DX)이 상승한다.

```jsx
// Controller
@Post()
CreateUser(@Body() CreateUserData: CreateUserDto){
	return this.userService.createUser(createUserData)
}

@Patch('/:id')
UpdateUserById(@Param('id') id: number, updateData: UpdateUserDto){
	return this.userService.updateUserById(id, updateData)
}
```

또한 확장하여 이것을 dataValidator로 사용할 수도 있다. (유효성 검증).

유효성 검증 기능을 추가하기 위해서는 `pipe` 를 작성해야한다. 미들웨어같은 개념이다. 클래스를 검증하는 npm 라이브러리도 함께 설치한다.

> npm install class-validator class-transformer

```jsx
// main.ts

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
}
```

ValidationPipe의 옵션({})으로 좀 더 높은 보안을 설정할 수도 있고, transform : true 옵션은 request시에 받는 parameter를 controller parameter가 정한 타입으로 변경해준다.👍

이후에 DTO를 다음과 같이 작성한다.

```jsx
// /dto/update-user.dto.ts

import { IsString, IsNumber } from 'class-validator'

export class CreateUserDto{
	@IsString()
	readonly name: string
	@IsNumber()
	readonly age : number
	@IsOptional()
	@IsString({ each : true })
	readonly alias : string[]
}

```

개발 경험 향상과 더불어 DTO는 유효하지 않은 request에 대해서 적절한 client error response를 반환한다.

class-validator는 이외에도 다양한 데코레이터를 지원한다.

> npm install @nestjs/mapped-types

Update method와 Create method의 차이는 각 field가 필수인지 아닌지 밖에 없으므로 코드의 반복을 피하기 위해서 위의 라이브러리를 설치하고, 다음과 같이 DTO를 작성할 수 있다.

```jsx
export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

# Service

> nest generate service

Controller에서 String을 반환하여 View를 그린다면 Service는 왜 필요할까?

결론적으로 Service에는 Controller가 실행하는 함수의 Logic을 작성한다. 함수의 logic은 SRP(Single Responsibility Principle) 을 준수한다.

## 구조와 아키텍쳐

NestJS의 바람 → Controller와 Business logic을 구분한다. 즉, Controller는 url을 가져오고, 함수를 실행하는 역할만 한다. 함수의 로직은 서비스로 가는 것이 바람직하다.

```jsx
// Controller
@Get('/hello')
sayHelloWorld(): string{
	this.appService.sayHelloworld()
}

// 위 예시를 NestJS의 아키텍쳐로 변환하면

//Service
@Injectable
export class UserService{
	...
  sayHelloWorld(): string {
		return 'Hello world'
	}
}
```

Controller의 함수이름과 Service의 함수이름이 반드시 같아야 할 필요는 없다.

따라서 데이터베이스와 통신을 하는 경우에는 Controller가 아닌 Service에 작성하는 것이 바람직하다.

## Entity

> user.entity.ts

Service로 보내고 받는 클래스(Interface)를 작성한다. 실제 데이터베이스의 모델이다.

```jsx
// /entity/user.entity.ts
export class User {
  id: number;
  name: string;
  alias: string[];
}
```

Controller의 로직을 전부 `this.UserService.메소드` 로 바꾼다는 전제하에 위의Controller, Entity와 연계해서 Service를 작성해보면,

```jsx
@Injectable()
export class UserService{
	private users: User[] = []

	getUsers(): User[] {
		return this.users
	}

	getUserById(id : number): User{
		const user = this.users.find((user)=> user.id === id)
		if(!user) throw new NotFoundException('No user found')
		return user
	}
}
```

반드시 Controller 내부에 `constructor(private readonly userService: UserService){}`를 작성해야한다.

흥미로운 점은 `getUserById` 에서 id에 해당하는 user가 존재하지 않을 때 nestJS 내장 기능인 `notFoundException` 을 사용할 수 있다는 것이다. delete나 patch 메소드를 사용할 때 this.getUserById를 먼저 실행시키고 `notFoundException` 을 통과하는 경우에만 정상 응답을 받을 수 있다. 에러를 던지는 경우 자동적으로 예외처리와 함께 지정한 msg도 함께 응답된다.
