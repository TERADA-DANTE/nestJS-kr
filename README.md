# main.ts

이름을 변경할 수 없다.

`AppModule` 로 앱을 실행시킨다.

# AppModule

Module : 애플리케이션의 일부분, Django에서의 App

Root module

Ex) 인증을 담당하는 애플리케이션 → User Module

# Controller

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

# Service

Controller에서 String을 반환하여 View를 그린다면 Service는 왜 필요할까?

결론적으로 Service에는 Controller가 실행하는 함수의 Logic을 작성한다.

## 구조와 아키텍쳐

NestJS의 바람 → Controller와 Business logic을 구분한다. 즉, Controller는 url을 가져오고, 함수를 실행하는 역할만 한다. 함수의 로직은 서비스로 가는 것이 바람직하다.

```jsx
// 위 예시를 NestJS의 아키텍쳐로 변환하면

// Controller
@Get('/hello')
sayHelloWorld(): string{
	this.appService.sayHelloworld()
}

//Service
@Injectable
export class AppService{
	...
  sayHelloWorld(): string {
		return 'Hello world'
	}
}
```

Controller의 함수이름과 Service의 함수이름이 반드시 같아야 할 필요는 없다.

따라서 데이터베이스와 통신을 하는 경우에는 Controller가 아닌 Service에 작성하는 것이 바람직하다.
