# NestJS 커스텀 데코레이터 가이드

## User 커스텀 데코레이터

NestJS에서는 커스텀 데코레이터를 만들어 코드를 더 깔끔하고 재사용 가능하게 만들 수 있습니다. 이 문서에서는 `User` 커스텀 데코레이터의 구현과 사용법에 대해 설명합니다.

### User 데코레이터 구현

`user.decorator.ts` 파일에서 `User` 데코레이터는 다음과 같이 구현되어 있습니다:

```typescript
import { createParamDecorator, ExecutionContext, InternalServerErrorException } from "@nestjs/common";
import { UsersModel } from "../entities/users.entity";

export const User = createParamDecorator((data: keyof UsersModel | undefined, context: ExecutionContext) => {
    const req = context.switchToHttp().getRequest();

    const user = req.user as UsersModel;
    if(!user){
        throw new InternalServerErrorException('User데코레이터는 AccessTokenGuard와 함께 사용해야 합니다. Request에 user 프로퍼티가 존재하지 않습니다.');
    }
    if(data){
        return user[data];
    }
    return user;
})
```

### 주요 특징

1. **파라미터 데코레이터**: `createParamDecorator`를 사용하여 메서드의 파라미터에 적용할 수 있는 데코레이터를 생성합니다.

2. **타입 안전성**: `data` 파라미터는 `keyof UsersModel | undefined` 타입으로 정의되어 있어, `UsersModel`의 프로퍼티 이름만 전달할 수 있도록 타입 안전성을 보장합니다.

3. **요청 객체 접근**: `context.switchToHttp().getRequest()`를 통해 HTTP 요청 객체에 접근합니다.

4. **사용자 정보 추출**: 요청 객체에서 `user` 프로퍼티를 추출하고, 이를 `UsersModel` 타입으로 캐스팅합니다.

5. **에러 처리**: 사용자 정보가 없는 경우 `InternalServerErrorException`을 발생시킵니다.

6. **선택적 프로퍼티 반환**: `data` 파라미터가 제공된 경우, 사용자 객체의 해당 프로퍼티만 반환합니다. 그렇지 않으면 전체 사용자 객체를 반환합니다.

## 컨트롤러에서의 사용법

`posts.controller.ts` 파일에서 `User` 데코레이터는 다음과 같이 사용됩니다:

```typescript
@Post()
@UseGuards(AccessTokenGuard)
postPosts(
  @User('id') userId: number,
  @Body('title') title: string,
  @Body('content') content: string,
){
  return this.postsService.createPost(userId, title, content);
};
```

### 사용 패턴

1. **가드와 함께 사용**: `@UseGuards(AccessTokenGuard)`와 함께 사용하여 인증된 요청에서만 사용자 정보에 접근할 수 있도록 합니다.

2. **특정 프로퍼티 추출**: `@User('id')`와 같이 사용하여 사용자 객체에서 특정 프로퍼티만 추출할 수 있습니다.

3. **타입 안전성**: 추출된 프로퍼티는 적절한 타입(`userId: number`)으로 선언됩니다.

## 장점

1. **코드 간소화**: 컨트롤러 메서드에서 매번 `req.user`를 통해 사용자 정보에 접근하는 대신, 데코레이터를 사용하여 간결하게 코드를 작성할 수 있습니다.

2. **재사용성**: 여러 컨트롤러에서 동일한 패턴으로 사용자 정보에 접근할 수 있습니다.

3. **타입 안전성**: TypeScript의 타입 시스템을 활용하여 타입 안전성을 보장합니다.

4. **유지보수성**: 사용자 정보 접근 로직이 한 곳에 집중되어 있어 유지보수가 용이합니다.

## 주의사항

1. `User` 데코레이터는 반드시 `AccessTokenGuard`와 함께 사용해야 합니다. 그렇지 않으면 `req.user` 프로퍼티가 존재하지 않아 예외가 발생합니다.

2. 데코레이터에 전달하는 프로퍼티 이름은 `UsersModel`에 존재하는 프로퍼티여야 합니다.

## 결론

NestJS의 커스텀 데코레이터는 코드의 재사용성과 가독성을 높이는 강력한 도구입니다. `User` 데코레이터를 사용하면 인증된 사용자의 정보에 쉽게 접근할 수 있으며, 특정 프로퍼티만 추출하여 사용할 수도 있습니다.
