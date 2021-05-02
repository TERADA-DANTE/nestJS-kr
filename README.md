# 시작하기에 앞서

이 문서는 개인의 공부와 연구, 그리고 [NestJS 공식문서](https://docs.nestjs.com/) 를 기반으로 작성하였습니다. 입문자에게 너무 깊은 내용은 과감하게 삭제하였습니다. 오탈자를 비롯한 정정 등의 Contribute는 언제나 환영합니다. 🐳

# 개요

> npm i -g @nestjs/cli

NestJS는 Node.js 서버 사이드 애플리케이션을 구축하기 위한 **프레임워크**입니다.

## Why NestJS?

NodeJS기반 백엔드라고 물으면 누구나 Express를 대답하겠죠? 저도 그렇게 생각해왔었습니다. Express의 철학에는 **자유로움**이라는 가치가 중요시되는데, 이런한 점이 많은 개발자들을 매료시켰다고 생각합니다.

하지만... 귀찮아요. 타입스크립트 설정하고, 린트 설정하고, 뭐 좀 하려고하면 무조건 npm install ...

한편, 좋아하는 프레임워크 중 하나인 Next.js 는 API기능을 가지고 있지만, 사실 정적 페이지 생성 프레임워크에 포함되어있는 **덤**이라고 생각하고 있습니다. 유효성 검증이나, 변환 등의 지원도 미미하고, 응답이나 예외처리 부분에서도 상당히 손이 많이갑니다. 버전 12정도 되면 API지원도 괜찮아질까요?

그렇기 때문에 NestJS입니다. 러닝커브가 좀 있을 수 있고, 비교적 강렬한 철학에 따라야한다는 제한은 있지만, 자유로움이 없어진 만큼 **규칙**이 있습니다. 굉장히 도전적인 스택이라고 생각합니다. 미래를 이끌 혁신적인 프레임워크라고 생각되진 않지만 자바스크립트 엔지니어로서 알고있으면 나쁘지않은 **백엔드** 프레임워크입니다.

# main.ts

이름을 변경할 수 없습니다

`AppModule` 로 앱을 실행시킵니다.

# Module

> nest generate module

모듈은 `@Module()` 데코레이터로 주석이 달린 클래스입니다. `@Module()` 데코레이터는 Nest가 응용 프로그램 구조를 구성하는 데 사용하는 메타 데이터를 제공합니다.

Module : 애플리케이션의 일부분, Django에서의 App

AppModule : main.ts 가 참조하는 Root module.

AppModule은 AppController와 AppService 이외를 가질 수 없습니다. 즉 회원 등록, 열람, 수정, 삭제기능을 하는 User 모듈이 존재한다고 하면 다음과 같이 작성됩니다.

```jsx
main.ts <- AppModule

// app.module.ts
@Module({
	imports : [UserModule],
	controllers : [AppController],
	provider : [AppService]
})

// users.module.ts
@Module({
	// 다른 모듈에서 동일한 Provider의 인스턴스를 공유하고 싶을때 import
	imports : [...]
	controllers : [UsersController],
	providers : [UsersService]
	// 다른 모듈에서 동일한 Provider의 인스턴스를 공유하고 싶을때 export
	exports : [UsersService]
})

export class UsersModule {}
```

provider에 Service를 추가함으로써, Controller에서 Service의 import없이 타입 추가만으로 잘 작동하는 것을 알 수 있습니다. 이것을 Dependency Injection이라고 합니다. Service에 있는 데코레이터 `@Injectable` 가 이것을 의미합니다.

# Controller

> nest generate controller

컨트롤러는 들어오는 request를 처리하고 response를 반환합니다. 컨트롤러는 클래스와 데코레이터로 구성됩니다.

express에서 router같은 존재로, url을 가져오고 함수를 실행합니다.

Ex) @Get 데코레이터는 express의 get 라우터와 같은 역할

```jsx
@Get('/hello')
sayHelloWorld() : string {
	return 'Hello world'
}

// localhost:3000/hello -> Hello world
```

데코레이터는 데코레이터의 함수나 클래스와 붙어있어야 합니다. 개행이 들어가면 작동할 수 없습니다.

여기에서 `sayHelloWorld()` 라는 메소드명은 아무런 의미도 가지지 않습니다.

이외에도 `Post` , `Delete`, `Put` , `Patch` 등도 같은 방식으로 작성한다. 파라미터를 받는 방법에 대해서는 아래에서 설명한다.

PUT 메소드는 모든 리소스를 업데이트하기 때문에 일부 리소스를 업데이트하는 PATCH 메소드를 사용하는 것이 바람직합니다.

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
		return `A user profile specified by ${id}`
	}

	@Patch('/:id')
	updateUserById(@Param() params, @Body() userData){
		return `A user profile specified by ${params.id} which has updated`
	}
}
```

위의 경우 `@Get()` 가 `'/'` 을 라우팅합니다. 하지만 `@Controller('/users')` 는 `'/users'` 스코프를 의미하기 때문에(엔트리 포인트) 결국 `[localhost:3000/users](http://localhost:3000/foo)` 로 라우팅이 생성됩니다.

### Route parameters

`@Param('id') id: number` 에서 `@Param('id')` 는 url에서 id가 파라미터임을 지시해줍니다. 이후 받아온 파라미터를 이하 logic에서 id라는 변수(`string`)에 담아 사용합니다. 즉 변수 이름은 변경할 수 있습니다.

즉,`'/:id'` 는 그 자체로는 로직안에서 파라미터로 작용할 수 없고, 오로지 다이나믹 라우팅 만을 담당합니다.

Restful API 를 작성하기 위해서는 Controller의 이름을 복수형으로 할 필요가 있습니다.

request 의 body를 받는 경우에는 `@Body() reqBody : type` 의 형식으로 작성합니다. NestJS에서는 JSON을 string으로 변환하거나 역변환하는 설정이 필요없습니다.

`/serach?name=uniqueName` 과 같은 방식의 get params의 라우팅은 다음과 같습니다.

```jsx
@Get('/search')
search(@Query("name"), userName : string){
	return 'A user'
}

// 혹은
@Get('/search')
search(@Query(), query){
	return 'A user'
}
```

### Routes order

위의 코드가 `Get('/:id')` 보다 아래에 있다면 이것은 `'/:id'` 로 인식되어 작동하지 않습니다. Get(Patch)과 Post가 각각 `@Query(parameter)` , `@Body()` 라는 것에 주의합니다.

### Route wildcards

패턴 기반 경로도 지원합니다. 예를 들어 별표(`*`)는 와일드 카드로 사용되며 모든 문자 조합과 일치합니다.

```jsx
@Get('/ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```

`'/ab*cd'`라우트 경로는 `abcd`, `ab_cd`, `abecd` 등과 일치합니다. `?`, `+` 및 `()`문자는 라우트 경로에 사용될 수 있으며 정규 표현식의 하위 집합입니다. 하이픈 (`-`)과 점 (`.`)은 문자 그대로 문자열 기반 경로로 해석됩니다.

### Status code

Status code 는 **201**인 POST 요청을 제외하고 기본적으로 항상 **200**입니다. 핸들러 레벨에서 `@HttpCode` 데코레이터를 사용하여 변경할 수 있습니다.

```jsx
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

## Express

NestJS는 Express위에서 동작하고 있습니다. 즉, 필요에 따라 express의 request, response에 접근할 수 있습니다.

```jsx
@Get()
GetUserById(@Req() request, @Res() response){}
```

그럼에도 불구하고 위와 같은 방식은 추천되지 않습니다.

이유는 NestJS가 fastify와 호환되는 기능을 가지고 있기 때문입니다. NestJS는 express와 fastify 두 개의 Framework 위에서 동작하기 때문에, Express의 객체에 직접적으로 접근하는 것은 미래를 생각하지 않는 개발 습관입니다.

## DTO

데이터 모델은 Entity(아래에 있음) 에서 관리한다. 그런데, Patch나 Create 메소드의 경우 body로 들어오는 데이터의 모델은 어떻게 할까요?

DTO는 데이터 전송 객체(Data Transfer Object)로, 다음과 같습니다.

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

이렇게 만들어진 DTO는 Controller와 Service에서 type interface처럼 사용할 수 있습니다. [DTO와 Interface는 엄연히 구분됩니다.](https://stackoverflow.com/questions/53531488/nestjs-why-do-we-need-dtos-and-interfaces-both-in-nestjs) DTO는 Network상에서 데이터가 어떻게 전달될지를 정의합니다. 라우팅 레벨에서 Controller의 메소드가 실행되기 전 단계에서 DTO는 Servie의 interface와 그 의미를 달리합니다.

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

### Pipe

DTO는 유효성 검증에도 사용 될 수 있습니다.

파이프는 일반적으로 2가지 케이스가 있습니다. 첫번째는 Transformation이고, 두번째는 Validation입니다. Validation(유효성 검증 기능)을 추가하기 위해서는 `pipe` 를 작성해야합니다. NestJS는 메소드가 호출되기 직전에 파이프를 삽입하고 파이프는 메소드로 향하는 인수를 수신합니다.

파이프를 사용하기 위해서는 파이프의 인스턴스를 적절한 컨텍스트에 삽입해야합니다.

```jsx
@Get('/:id')
async findOne(@Param('id', parseIntPipe) id: number){
	return this.userService.findOne(id)
}
```

파이프는 메소드가 실행되기 전에 transform을 수행합니다. 따라서 userService의 number type의 id가 전달되거나, 적절한 에러를 반환할 것입니다. 위의 예제에서는 파이프의 클래스를 작성했습니다. 인스턴스를 작성하는 경우에는 더 많은 커스터마이제이션이 가능합니다.

```jsx
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.catsService.findOne(id);
}
```

아래는 파이프로 클래스를 검증하는 방법에 대해서 입니다.

클래스를 검증하는 npm 라이브러리는 다음과 같습니다.

> npm install class-validator class-transformer

```jsx
// main.ts
// 파이프를 글로벌에 등록합니다.

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
}
```

ValidationPipe의 옵션({})으로 좀 더 높은 보안을 설정할 수도 있고, transform : true 옵션은 request시에 받는 parameter를 controller parameter가 정한 타입으로 변경해줍니다.👍

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

개발 경험 향상과 더불어 DTO는 유효하지 않은 request에 대해서 적절한 client error response를 반환합니다.

class-validator는 이외에도 다양한 데코레이터를 지원합니다.

> npm install @nestjs/mapped-types

Update method와 Create method의 차이는 각 field가 필수인지 아닌지 밖에 없으므로 코드의 반복을 피하기 위해서 위의 라이브러리를 설치하고, 다음과 같이 DTO를 작성할 수 있습니다.

```jsx
export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

# Provider

Provider는 Nest의 기본 개념입니다. 많은 기본 Nest 클래스는 Service, repository, factory, helper 등으로 취급될 수 있습니다. Provider의 주요 아이디어는 의존성을 **주입**한다는 것입니다. 즉, 개체가 서로 다양한 관계를 맺을 수 있으며 개체의 **배선** 기능이 Nest 런타임 시스템에 위임됩니다.

위의 예제에는 UserController가 존재합니다. 컨트롤러에서 Logic을 전부 처리하는 것은 SOLID원칙에도 어긋나며, 아름다운 설계가 아니기 때문에 복잡한 작업(HTTP 요청을 처리하는 등)을 Provider에 위임해야합니다. Provider는 클래스 선언 앞에 `@Injectable` 데코레이터가 존재하는 클래스입니다. `@Injectable` 데코레이터는 메타 데이터를 첨부하여 이 클래스가 Provider임을 알려줍니다. 아래에서도 작성하겠지만, **주입**은 Controller 내부의 contructor에서 이루어집니다.

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

Controller의 함수이름과 Service의 함수이름이 반드시 같아야 할 필요는 없습니다.

따라서 데이터베이스와 통신을 하는 경우에는 Controller가 아닌 Service에 작성하는 것이 바람직합니다.

## Entity

> user.entity.ts

Service로 보내고 받는 클래스(Interface)를 작성합니다.

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

Controller 내부에 작성된 `constructor(private readonly userService: UserService){}`가 위의 Service의 인스턴스를 생성합니다. (Dependency Injection)

### Exception filters

NestJS는 어플리케이션 전반에서 핸들링 할 수 없는 에러를 만났을 때를 대비하여 exceptions layer를 내장하고 있습니다. 애플리케이션 코드 레벨에서 예외처리를 할 수 없을 때, 이 레이어가 그것을 포착하고 유저친화적인 응답을 보냅니다. 이는 파이프보다 선행되는 레이어입니다.

이를 global exception filter라고 하는데, 이것의 타입은 `HttpException`입니다. `HttpException` 클래스가 아니거나, 상속받은 클래스가 아니라면 내장 필터는 다음과 같은 default 응답을 보입니다.

```jsx
{
	"statusCode" : 500,
  "message" : "Internal server error"
}
```

`@nestjs/common` 으로부터 이 클래스를 가져와서 다음과 같이 instance와 함께 사용자 지정 예외처리를 할 수 있습니다.

```jsx
// users.controller.ts
@Get('/error')
error(){
	throw new HttpException(response, status)
}
```

response는 문자열 혹은 객체가 될 수 있는데, 이 문서에서는 문자열인 경우만 다루겠습니다. 문자열을 지정하면 해당 문자열이 응답의 body message로써 작성됩니다. status는 일반적으로 내장된 HttpStatus객체를 사용하고 이를 이용해 http status code를 지정합니다.

위에서 잠깐 언급한 `HttpException` 을 상속한 예외 처리 클래스가 NestJS는 상당히 많이 내장되어있습니다. 예를 들어 예제의 `getUserById` 에서 id에 해당하는 user가 존재하지 않을 때 nestJS 내장 기능인 `notFoundException` 을 사용할 수 있습니다. 이로써 delete나 patch 메소드를 사용할 때 this.getUserById를 먼저 실행시키고 `notFoundException` 을 통과하는 경우에만 정상 응답을 받을 수 있습니다.

exception 개념안에는 `@useFilter()` 를 사용하는 filter의 개념도 있는데, filter는 다양한 인풋에 대해서 로그를 남기거나, 전혀다른 JSON 스키마를 응답하는 등의 exception layer 전반의 풀 컨트롤을 제공합니다. 이것에 관해서는 추후에 정리하겠습니다.

# Middleware

> nest generate middleware

미들웨어는 라우트 핸들러가 실행되기 **전**에 실행됩니다. 미들웨어는 `request` 와 `response` 그리고 어플리케이션의 request-response cycle의 `next()` 에 접근할 수 있습니다.

기본적으로 Nest의 미들웨어는 express의 미들웨어와 같습니다. 다음의 5가지 항목이 Express의 공식문서에서 말하는 미들웨어의 기능입니다.

```jsx
미들웨어는
- 코드를 실행합니다.
- request와 response를 변경할 수 있습니다.
- reqeust-response cycle을 종료할 수 있습니다.
- stack 내부에서 next 미들웨어 함수를 부를 수 있습니다.
- 해당 미들웨어가 request-response cycle을 종료시키지 않는다면 next() 가 명시적으로 선언되어야 합니다. 그렇지않으면 request는 계속 걸리게(hanging) 될 것입니다.
```

미들웨어는 함수의 형태, 혹은 `@Injectable` 데코레이터와 함께 클래스의 형태로 선언됩니다. 함수는 특별한 제약이 없지만 클래스의 경우 `NestMiddleware` 인터페이스를 implements 해야 합니다.

request와 response를 console.log하는 간단한 미들웨어를 작성하겠습니다.

```jsx
// /users/middleware/users.middleware.ts

@Injectable()
export class UsersMiddleware implements NextMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(req, res);
    next();
  }
}
```

미들웨어를 등록하는 방법은 다음과 같습니다.

```jsx
// /users/users.module.ts

@Module({
	...
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(UsersMiddleware)
      .forRoutes('users')
  }
}
```

`forRoutes` 가 주목할만한 부분입니다. 문자열을 입력하면 해당 라우트의 모든 메소드에 미들웨어가 적용됩니다. 다음과 같이 메소드를 특정할 수 도 있습니다.

```jsx
consumer
  .apply(UsersMiddleware)
  .forRoutes({ path: "users", method: RequestMethod.GET });
```

이외에도, 특정 라우트를 제외하거나, 라우트에 와일드카드를 적용하는 등의 방법이 있습니다. 복수의 미들웨어를 적용할 때에는 apply의 인수에 추가할 수 있습니다.

물론 forRoutes가 다른경우에는 consumer를 여러개 작성해야하겠지만, 이 점은 성숙하지 못한 프레임워크의 한계라고 생각하면 마음이 편할 것 같습니다.

글로벌하게 미들웨어를 등록하는 경우는 다음과 같습니다..

```jsx
const app = await NestFactory.create(AppModule);
app.use(UsersMiddleware);
await app.listen(3000);
```
