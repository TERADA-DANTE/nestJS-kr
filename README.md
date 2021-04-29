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
