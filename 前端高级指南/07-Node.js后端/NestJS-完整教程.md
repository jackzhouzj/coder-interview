# NestJS - 完整教程

## 课程简介

NestJS是一个用于构建高效、可扩展的Node.js服务器端应用的框架。它使用渐进式JavaScript,完全支持TypeScript,结合了OOP(面向对象编程)、FP(函数式编程)和FRP(函数响应式编程)的元素。本教程将深入讲解NestJS的核心概念、架构设计、依赖注入、模块系统等内容。

## 学习目标

- 理解NestJS的设计理念和架构
- 掌握装饰器的使用
- 熟练使用依赖注入
- 掌握模块化开发
- 学会控制器和服务的编写
- 理解中间件、管道、守卫、拦截器
- 掌握数据库集成
- 学会构建微服务
- 掌握测试和部署

## 目录

1. [NestJS基础](#第1章-nestjs基础)
2. [装饰器系统](#第2章-装饰器系统)
3. [依赖注入](#第3章-依赖注入)
4. [模块系统](#第4章-模块系统)
5. [控制器](#第5章-控制器)
6. [服务提供者](#第6章-服务提供者)
7. [中间件](#第7章-中间件)
8. [管道与验证](#第8章-管道与验证)
9. [守卫与授权](#第9章-守卫与授权)
10. [拦截器](#第10章-拦截器)

---

## 第1章 NestJS基础

### 1.1 安装与初始化

```bash
# 安装NestJS CLI
npm install -g @nestjs/cli

# 创建新项目
nest new my-nest-app

# 进入项目目录
cd my-nest-app

# 启动开发服务器
npm run start:dev
```


```typescript
/**
 * 第一个NestJS应用 - main.ts
 * @author erik.zhou
 */

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    await app.listen(3000);
    console.log(`应用运行在: ${await app.getUrl()}`);
}
bootstrap();
```

```typescript
/**
 * 应用模块 - app.module.ts
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
    imports: [],
    controllers: [AppController],
    providers: [AppService],
})
export class AppModule {}
```

```typescript
/**
 * 应用控制器 - app.controller.ts
 * @author erik.zhou
 */

import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
    constructor(private readonly appService: AppService) {}

    @Get()
    getHello(): string {
        return this.appService.getHello();
    }
}
```

```typescript
/**
 * 应用服务 - app.service.ts
 * @author erik.zhou
 */

import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
    getHello(): string {
        return 'Hello NestJS!';
    }
}
```


### 1.2 项目结构

```
my-nest-app/
├── src/
│   ├── app.controller.ts      # 控制器
│   ├── app.controller.spec.ts # 控制器测试
│   ├── app.module.ts          # 根模块
│   ├── app.service.ts         # 服务
│   └── main.ts                # 入口文件
├── test/                      # 测试文件
├── nest-cli.json              # NestJS CLI配置
├── package.json
├── tsconfig.json              # TypeScript配置
└── tsconfig.build.json
```

```typescript
/**
 * NestJS项目结构最佳实践
 * @author erik.zhou
 */

// 推荐的项目结构
/*
src/
├── common/                    # 公共模块
│   ├── decorators/           # 自定义装饰器
│   ├── filters/              # 异常过滤器
│   ├── guards/               # 守卫
│   ├── interceptors/         # 拦截器
│   ├── pipes/                # 管道
│   └── middleware/           # 中间件
├── config/                   # 配置文件
│   ├── database.config.ts
│   └── app.config.ts
├── modules/                  # 业务模块
│   ├── users/
│   │   ├── dto/             # 数据传输对象
│   │   ├── entities/        # 实体
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   └── users.module.ts
│   └── posts/
│       ├── dto/
│       ├── entities/
│       ├── posts.controller.ts
│       ├── posts.service.ts
│       └── posts.module.ts
├── app.module.ts
└── main.ts
*/
```

### 1.3 配置与环境变量

```typescript
/**
 * 环境变量配置
 * @author erik.zhou
 */

// 安装依赖
// npm install @nestjs/config

import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
    imports: [
        ConfigModule.forRoot({
            isGlobal: true,
            envFilePath: '.env',
            ignoreEnvFile: process.env.NODE_ENV === 'production',
        }),
    ],
})
export class AppModule {}
```

```typescript
/**
 * 使用配置服务
 * @author erik.zhou
 */

import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class AppService {
    constructor(private configService: ConfigService) {}

    getDatabaseConfig() {
        return {
            host: this.configService.get<string>('DATABASE_HOST'),
            port: this.configService.get<number>('DATABASE_PORT'),
            username: this.configService.get<string>('DATABASE_USER'),
            password: this.configService.get<string>('DATABASE_PASSWORD'),
        };
    }
}
```


### 1.4 生命周期钩子

```typescript
/**
 * 生命周期钩子
 * @author erik.zhou
 */

import {
    Injectable,
    OnModuleInit,
    OnModuleDestroy,
    OnApplicationBootstrap,
    OnApplicationShutdown,
} from '@nestjs/common';

@Injectable()
export class AppService
    implements
        OnModuleInit,
        OnApplicationBootstrap,
        OnModuleDestroy,
        OnApplicationShutdown
{
    onModuleInit() {
        console.log('1. 模块初始化');
    }

    onApplicationBootstrap() {
        console.log('2. 应用启动完成');
    }

    onModuleDestroy() {
        console.log('3. 模块销毁');
    }

    onApplicationShutdown(signal?: string) {
        console.log('4. 应用关闭', signal);
    }
}
```

```typescript
/**
 * 优雅关闭
 * @author erik.zhou
 */

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    
    // 启用优雅关闭
    app.enableShutdownHooks();
    
    await app.listen(3000);
    
    // 监听关闭信号
    process.on('SIGTERM', async () => {
        console.log('收到SIGTERM信号');
        await app.close();
    });
}
bootstrap();
```

---

## 第2章 装饰器系统

### 2.1 类装饰器

```typescript
/**
 * 类装饰器
 * @author erik.zhou
 */

import { Controller, Injectable, Module } from '@nestjs/common';

// @Controller装饰器
@Controller('users')
export class UsersController {
    // 控制器逻辑
}

// @Injectable装饰器
@Injectable()
export class UsersService {
    // 服务逻辑
}

// @Module装饰器
@Module({
    controllers: [UsersController],
    providers: [UsersService],
    exports: [UsersService],
})
export class UsersModule {}
```


### 2.2 方法装饰器

```typescript
/**
 * 方法装饰器
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    Post,
    Put,
    Delete,
    Patch,
    HttpCode,
    Header,
    Redirect,
} from '@nestjs/common';

@Controller('users')
export class UsersController {
    // GET请求
    @Get()
    findAll() {
        return { users: [] };
    }

    // POST请求
    @Post()
    @HttpCode(201)
    create() {
        return { message: '创建成功' };
    }

    // PUT请求
    @Put(':id')
    update() {
        return { message: '更新成功' };
    }

    // DELETE请求
    @Delete(':id')
    @HttpCode(204)
    remove() {
        return;
    }

    // PATCH请求
    @Patch(':id')
    partialUpdate() {
        return { message: '部分更新成功' };
    }

    // 设置响应头
    @Get('with-header')
    @Header('Cache-Control', 'no-cache')
    getWithHeader() {
        return { data: 'cached' };
    }

    // 重定向
    @Get('redirect')
    @Redirect('https://nestjs.com', 301)
    redirect() {
        return;
    }
}
```

### 2.3 参数装饰器

```typescript
/**
 * 参数装饰器
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    Post,
    Param,
    Query,
    Body,
    Headers,
    Ip,
    HostParam,
    Req,
    Res,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Controller('users')
export class UsersController {
    // 路径参数
    @Get(':id')
    findOne(@Param('id') id: string) {
        return { userId: id };
    }

    // 多个路径参数
    @Get(':userId/posts/:postId')
    findPost(
        @Param('userId') userId: string,
        @Param('postId') postId: string,
    ) {
        return { userId, postId };
    }

    // 查询参数
    @Get()
    findAll(
        @Query('page') page: number = 1,
        @Query('limit') limit: number = 10,
    ) {
        return { page, limit };
    }

    // 请求体
    @Post()
    create(@Body() createUserDto: any) {
        return { user: createUserDto };
    }

    // 请求头
    @Get('headers')
    getHeaders(@Headers('authorization') auth: string) {
        return { authorization: auth };
    }

    // IP地址
    @Get('ip')
    getIp(@Ip() ip: string) {
        return { ip };
    }

    // 完整请求对象
    @Get('request')
    getRequest(@Req() request: Request) {
        return {
            method: request.method,
            url: request.url,
        };
    }

    // 响应对象
    @Get('response')
    getResponse(@Res() response: Response) {
        response.status(200).json({ message: 'success' });
    }
}
```


### 2.4 自定义装饰器

```typescript
/**
 * 自定义装饰器
 * @author erik.zhou
 */

import { createParamDecorator, ExecutionContext } from '@nestjs/common';

// 获取当前用户装饰器
export const CurrentUser = createParamDecorator(
    (data: unknown, ctx: ExecutionContext) => {
        const request = ctx.switchToHttp().getRequest();
        return request.user;
    },
);

// 使用自定义装饰器
@Controller('profile')
export class ProfileController {
    @Get()
    getProfile(@CurrentUser() user: any) {
        return { user };
    }
}
```

```typescript
/**
 * 组合装饰器
 * @author erik.zhou
 */

import { applyDecorators, UseGuards, UseInterceptors } from '@nestjs/common';
import { AuthGuard } from './auth.guard';
import { LoggingInterceptor } from './logging.interceptor';

// 组合多个装饰器
export function Auth() {
    return applyDecorators(
        UseGuards(AuthGuard),
        UseInterceptors(LoggingInterceptor),
    );
}

// 使用组合装饰器
@Controller('admin')
export class AdminController {
    @Get()
    @Auth()
    getAdminData() {
        return { data: 'admin data' };
    }
}
```

```typescript
/**
 * 元数据装饰器
 * @author erik.zhou
 */

import { SetMetadata } from '@nestjs/common';

// 定义角色装饰器
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// 使用角色装饰器
@Controller('users')
export class UsersController {
    @Get()
    @Roles('admin', 'user')
    findAll() {
        return { users: [] };
    }

    @Post()
    @Roles('admin')
    create() {
        return { message: '创建成功' };
    }
}
```

---

## 第3章 依赖注入

### 3.1 基础依赖注入

```typescript
/**
 * 基础依赖注入
 * @author erik.zhou
 */

import { Injectable, Controller } from '@nestjs/common';

// 服务类
@Injectable()
export class UsersService {
    private users = [
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' },
    ];

    findAll() {
        return this.users;
    }

    findOne(id: number) {
        return this.users.find(user => user.id === id);
    }
}

// 控制器注入服务
@Controller('users')
export class UsersController {
    constructor(private readonly usersService: UsersService) {}

    @Get()
    findAll() {
        return this.usersService.findAll();
    }

    @Get(':id')
    findOne(@Param('id') id: string) {
        return this.usersService.findOne(+id);
    }
}
```


### 3.2 提供者作用域

```typescript
/**
 * 提供者作用域
 * @author erik.zhou
 */

import { Injectable, Scope } from '@nestjs/common';

// 默认作用域（单例）
@Injectable()
export class DefaultService {
    private counter = 0;

    increment() {
        return ++this.counter;
    }
}

// 请求作用域（每个请求创建新实例）
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {
    private counter = 0;

    increment() {
        return ++this.counter;
    }
}

// 瞬态作用域（每次注入创建新实例）
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {
    private counter = 0;

    increment() {
        return ++this.counter;
    }
}
```

### 3.3 自定义提供者

```typescript
/**
 * 自定义提供者
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';

// 值提供者
const configProvider = {
    provide: 'CONFIG',
    useValue: {
        apiUrl: 'https://api.example.com',
        timeout: 5000,
    },
};

// 类提供者
const serviceProvider = {
    provide: 'UsersService',
    useClass: UsersService,
};

// 工厂提供者
const factoryProvider = {
    provide: 'DATABASE_CONNECTION',
    useFactory: async (configService: ConfigService) => {
        const config = configService.getDatabaseConfig();
        return await createConnection(config);
    },
    inject: [ConfigService],
};

// 别名提供者
const aliasProvider = {
    provide: 'AliasService',
    useExisting: UsersService,
};

@Module({
    providers: [
        configProvider,
        serviceProvider,
        factoryProvider,
        aliasProvider,
    ],
})
export class AppModule {}
```

### 3.4 可选依赖

```typescript
/**
 * 可选依赖
 * @author erik.zhou
 */

import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class UsersService {
    constructor(
        @Optional()
        @Inject('OPTIONAL_SERVICE')
        private optionalService?: any,
    ) {
        if (this.optionalService) {
            console.log('可选服务已注入');
        } else {
            console.log('可选服务未注入');
        }
    }

    doSomething() {
        if (this.optionalService) {
            return this.optionalService.execute();
        }
        return '使用默认行为';
    }
}
```

### 3.5 循环依赖

```typescript
/**
 * 循环依赖处理
 * @author erik.zhou
 */

import { Injectable, forwardRef, Inject } from '@nestjs/common';

@Injectable()
export class UsersService {
    constructor(
        @Inject(forwardRef(() => PostsService))
        private postsService: PostsService,
    ) {}

    getUserPosts(userId: number) {
        return this.postsService.findByUserId(userId);
    }
}

@Injectable()
export class PostsService {
    constructor(
        @Inject(forwardRef(() => UsersService))
        private usersService: UsersService,
    ) {}

    findByUserId(userId: number) {
        const user = this.usersService.findOne(userId);
        return [];
    }
}
```

---

## 第4章 模块系统

### 4.1 基础模块

```typescript
/**
 * 基础模块定义
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
    controllers: [UsersController],
    providers: [UsersService],
    exports: [UsersService],
})
export class UsersModule {}
```


### 4.2 功能模块

```typescript
/**
 * 功能模块 - users.module.ts
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';

@Module({
    imports: [TypeOrmModule.forFeature([User])],
    controllers: [UsersController],
    providers: [UsersService],
    exports: [UsersService],
})
export class UsersModule {}
```

### 4.3 共享模块

```typescript
/**
 * 共享模块
 * @author erik.zhou
 */

import { Module, Global } from '@nestjs/common';
import { DatabaseService } from './database.service';
import { LoggerService } from './logger.service';

// 全局模块
@Global()
@Module({
    providers: [DatabaseService, LoggerService],
    exports: [DatabaseService, LoggerService],
})
export class SharedModule {}
```

```typescript
/**
 * 使用共享模块
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { SharedModule } from './shared/shared.module';
import { UsersModule } from './users/users.module';

@Module({
    imports: [
        SharedModule, // 导入一次，所有模块可用
        UsersModule,
    ],
})
export class AppModule {}
```

### 4.4 动态模块

```typescript
/**
 * 动态模块
 * @author erik.zhou
 */

import { Module, DynamicModule } from '@nestjs/common';
import { DatabaseService } from './database.service';

export interface DatabaseModuleOptions {
    host: string;
    port: number;
    username: string;
    password: string;
}

@Module({})
export class DatabaseModule {
    static forRoot(options: DatabaseModuleOptions): DynamicModule {
        return {
            module: DatabaseModule,
            providers: [
                {
                    provide: 'DATABASE_OPTIONS',
                    useValue: options,
                },
                DatabaseService,
            ],
            exports: [DatabaseService],
        };
    }

    static forRootAsync(options: {
        useFactory: (...args: any[]) => Promise<DatabaseModuleOptions>;
        inject?: any[];
    }): DynamicModule {
        return {
            module: DatabaseModule,
            providers: [
                {
                    provide: 'DATABASE_OPTIONS',
                    useFactory: options.useFactory,
                    inject: options.inject || [],
                },
                DatabaseService,
            ],
            exports: [DatabaseService],
        };
    }
}
```

```typescript
/**
 * 使用动态模块
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { ConfigService } from '@nestjs/config';

@Module({
    imports: [
        // 同步配置
        DatabaseModule.forRoot({
            host: 'localhost',
            port: 3306,
            username: 'root',
            password: 'password',
        }),

        // 异步配置
        DatabaseModule.forRootAsync({
            useFactory: (configService: ConfigService) => ({
                host: configService.get('DB_HOST'),
                port: configService.get('DB_PORT'),
                username: configService.get('DB_USER'),
                password: configService.get('DB_PASSWORD'),
            }),
            inject: [ConfigService],
        }),
    ],
})
export class AppModule {}
```

### 4.5 模块重导出

```typescript
/**
 * 模块重导出
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { PostsModule } from './posts/posts.module';
import { CommentsModule } from './comments/comments.module';

// 核心模块，重导出其他模块
@Module({
    imports: [UsersModule, PostsModule, CommentsModule],
    exports: [UsersModule, PostsModule, CommentsModule],
})
export class CoreModule {}
```

```typescript
/**
 * 使用重导出模块
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { CoreModule } from './core/core.module';

@Module({
    imports: [
        CoreModule, // 一次导入，获得所有核心模块
    ],
})
export class AppModule {}
```

---

## 第5章 控制器

### 5.1 基础控制器

```typescript
/**
 * 基础控制器
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    Post,
    Put,
    Delete,
    Param,
    Body,
    Query,
} from '@nestjs/common';

@Controller('users')
export class UsersController {
    @Get()
    findAll(@Query('page') page: number = 1) {
        return {
            users: [],
            page: page,
        };
    }

    @Get(':id')
    findOne(@Param('id') id: string) {
        return {
            user: { id: id, name: 'John' },
        };
    }

    @Post()
    create(@Body() createUserDto: any) {
        return {
            message: '创建成功',
            user: createUserDto,
        };
    }

    @Put(':id')
    update(@Param('id') id: string, @Body() updateUserDto: any) {
        return {
            message: '更新成功',
            user: { id: id, ...updateUserDto },
        };
    }

    @Delete(':id')
    remove(@Param('id') id: string) {
        return {
            message: '删除成功',
            userId: id,
        };
    }
}
```


### 5.2 DTO数据传输对象

```typescript
/**
 * DTO定义 - create-user.dto.ts
 * @author erik.zhou
 */

import {
    IsString,
    IsEmail,
    IsInt,
    Min,
    Max,
    IsOptional,
    MinLength,
    MaxLength,
} from 'class-validator';

export class CreateUserDto {
    @IsString()
    @MinLength(3)
    @MaxLength(20)
    name: string;

    @IsEmail()
    email: string;

    @IsInt()
    @Min(18)
    @Max(100)
    age: number;

    @IsString()
    @MinLength(8)
    password: string;

    @IsOptional()
    @IsString()
    phone?: string;
}
```

```typescript
/**
 * 使用DTO
 * @author erik.zhou
 */

import { Controller, Post, Body } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('users')
export class UsersController {
    @Post()
    create(@Body() createUserDto: CreateUserDto) {
        return {
            message: '创建成功',
            user: createUserDto,
        };
    }

    @Put(':id')
    update(
        @Param('id') id: string,
        @Body() updateUserDto: UpdateUserDto,
    ) {
        return {
            message: '更新成功',
            user: { id: id, ...updateUserDto },
        };
    }
}
```

### 5.3 响应处理

```typescript
/**
 * 响应处理
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    Post,
    HttpCode,
    HttpStatus,
    Header,
    Redirect,
    Res,
} from '@nestjs/common';
import { Response } from 'express';

@Controller('response')
export class ResponseController {
    // 设置状态码
    @Post()
    @HttpCode(HttpStatus.CREATED)
    create() {
        return { message: '创建成功' };
    }

    // 设置响应头
    @Get('header')
    @Header('Cache-Control', 'no-cache')
    @Header('X-Custom-Header', 'value')
    getWithHeader() {
        return { data: 'cached' };
    }

    // 重定向
    @Get('redirect')
    @Redirect('https://nestjs.com', 301)
    redirect() {
        return;
    }

    // 动态重定向
    @Get('dynamic-redirect')
    dynamicRedirect() {
        return {
            url: 'https://nestjs.com',
            statusCode: 302,
        };
    }

    // 使用Response对象
    @Get('custom')
    custom(@Res() res: Response) {
        res.status(200).json({
            message: '自定义响应',
        });
    }
}
```

### 5.4 异常处理

```typescript
/**
 * 异常处理
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    Param,
    NotFoundException,
    BadRequestException,
    UnauthorizedException,
    ForbiddenException,
    InternalServerErrorException,
    HttpException,
    HttpStatus,
} from '@nestjs/common';

@Controller('users')
export class UsersController {
    @Get(':id')
    findOne(@Param('id') id: string) {
        const user = this.findUserById(id);

        if (!user) {
            throw new NotFoundException(`用户 ${id} 不存在`);
        }

        return user;
    }

    @Post()
    create(@Body() createUserDto: any) {
        if (!createUserDto.name) {
            throw new BadRequestException('用户名不能为空');
        }

        return { message: '创建成功' };
    }

    @Get('protected')
    protected() {
        const isAuthenticated = false;

        if (!isAuthenticated) {
            throw new UnauthorizedException('未授权');
        }

        return { data: 'protected data' };
    }

    @Delete(':id')
    remove(@Param('id') id: string) {
        const hasPermission = false;

        if (!hasPermission) {
            throw new ForbiddenException('无权限删除');
        }

        return { message: '删除成功' };
    }

    // 自定义异常
    @Get('custom-error')
    customError() {
        throw new HttpException(
            {
                status: HttpStatus.FORBIDDEN,
                error: '自定义错误消息',
            },
            HttpStatus.FORBIDDEN,
        );
    }

    private findUserById(id: string) {
        return null;
    }
}
```

### 5.5 版本控制

```typescript
/**
 * API版本控制
 * @author erik.zhou
 */

import { Controller, Get, Version } from '@nestjs/common';

@Controller('users')
export class UsersController {
    // V1版本
    @Get()
    @Version('1')
    findAllV1() {
        return {
            version: 'v1',
            users: [{ id: 1, name: 'John' }],
        };
    }

    // V2版本
    @Get()
    @Version('2')
    findAllV2() {
        return {
            version: 'v2',
            users: [
                {
                    id: 1,
                    name: 'John',
                    email: 'john@example.com',
                },
            ],
        };
    }

    // 多版本支持
    @Get('info')
    @Version(['1', '2'])
    getInfo() {
        return { info: 'available in v1 and v2' };
    }
}
```

```typescript
/**
 * 启用版本控制 - main.ts
 * @author erik.zhou
 */

import { NestFactory } from '@nestjs/core';
import { VersioningType } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);

    // URI版本控制
    app.enableVersioning({
        type: VersioningType.URI,
        defaultVersion: '1',
    });

    // Header版本控制
    // app.enableVersioning({
    //     type: VersioningType.HEADER,
    //     header: 'X-API-Version',
    // });

    // Media Type版本控制
    // app.enableVersioning({
    //     type: VersioningType.MEDIA_TYPE,
    //     key: 'v=',
    // });

    await app.listen(3000);
}
bootstrap();
```

---

## 第6章 服务提供者

### 6.1 基础服务

```typescript
/**
 * 基础服务
 * @author erik.zhou
 */

import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
    private users = [
        { id: 1, name: 'John', email: 'john@example.com' },
        { id: 2, name: 'Jane', email: 'jane@example.com' },
    ];

    findAll() {
        return this.users;
    }

    findOne(id: number) {
        return this.users.find(user => user.id === id);
    }

    create(user: any) {
        const newUser = {
            id: this.users.length + 1,
            ...user,
        };
        this.users.push(newUser);
        return newUser;
    }

    update(id: number, updateData: any) {
        const index = this.users.findIndex(user => user.id === id);
        if (index !== -1) {
            this.users[index] = { ...this.users[index], ...updateData };
            return this.users[index];
        }
        return null;
    }

    remove(id: number) {
        const index = this.users.findIndex(user => user.id === id);
        if (index !== -1) {
            this.users.splice(index, 1);
            return true;
        }
        return false;
    }
}
```


### 6.2 服务间依赖

```typescript
/**
 * 服务间依赖
 * @author erik.zhou
 */

import { Injectable } from '@nestjs/common';

@Injectable()
export class PostsService {
    constructor(private readonly usersService: UsersService) {}

    async findUserPosts(userId: number) {
        const user = await this.usersService.findOne(userId);
        if (!user) {
            throw new NotFoundException('用户不存在');
        }

        return {
            user: user,
            posts: [],
        };
    }

    async createPost(userId: number, postData: any) {
        const user = await this.usersService.findOne(userId);
        if (!user) {
            throw new NotFoundException('用户不存在');
        }

        return {
            id: 1,
            userId: userId,
            ...postData,
        };
    }
}
```

### 6.3 异步服务

```typescript
/**
 * 异步服务
 * @author erik.zhou
 */

import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
    async findAll(): Promise<any[]> {
        // 模拟异步操作
        return new Promise(resolve => {
            setTimeout(() => {
                resolve([
                    { id: 1, name: 'John' },
                    { id: 2, name: 'Jane' },
                ]);
            }, 1000);
        });
    }

    async findOne(id: number): Promise<any> {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                const user = { id: id, name: 'John' };
                if (user) {
                    resolve(user);
                } else {
                    reject(new NotFoundException('用户不存在'));
                }
            }, 500);
        });
    }

    async create(userData: any): Promise<any> {
        try {
            // 异步数据库操作
            const user = await this.saveToDatabase(userData);
            return user;
        } catch (error) {
            throw new InternalServerErrorException('创建用户失败');
        }
    }

    private async saveToDatabase(data: any): Promise<any> {
        return new Promise(resolve => {
            setTimeout(() => {
                resolve({ id: 1, ...data });
            }, 500);
        });
    }
}
```

---

## 第7章 中间件

### 7.1 函数式中间件

```typescript
/**
 * 函数式中间件
 * @author erik.zhou
 */

import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
    next();
}
```

### 7.2 类中间件

```typescript
/**
 * 类中间件
 * @author erik.zhou
 */

import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction) {
        const start = Date.now();

        res.on('finish', () => {
            const duration = Date.now() - start;
            console.log(
                `[${new Date().toISOString()}] ${req.method} ${req.url} - ${res.statusCode} - ${duration}ms`,
            );
        });

        next();
    }
}
```

### 7.3 应用中间件

```typescript
/**
 * 应用中间件
 * @author erik.zhou
 */

import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { UsersModule } from './users/users.module';

@Module({
    imports: [UsersModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        // 应用到所有路由
        consumer.apply(LoggerMiddleware).forRoutes('*');

        // 应用到特定路由
        consumer
            .apply(LoggerMiddleware)
            .forRoutes('users', 'posts');

        // 应用到特定HTTP方法
        consumer
            .apply(LoggerMiddleware)
            .forRoutes({ path: 'users', method: RequestMethod.GET });

        // 排除特定路由
        consumer
            .apply(LoggerMiddleware)
            .exclude(
                { path: 'users', method: RequestMethod.POST },
                'users/(.*)',
            )
            .forRoutes(UsersController);
    }
}
```

### 7.4 多个中间件

```typescript
/**
 * 多个中间件
 * @author erik.zhou
 */

import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class AuthMiddleware implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction) {
        const token = req.headers.authorization;
        if (!token) {
            throw new UnauthorizedException('未授权');
        }
        next();
    }
}

@Injectable()
export class CorsMiddleware implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction) {
        res.header('Access-Control-Allow-Origin', '*');
        res.header('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE');
        next();
    }
}
```

```typescript
/**
 * 应用多个中间件
 * @author erik.zhou
 */

import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';

@Module({
    imports: [UsersModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        consumer
            .apply(LoggerMiddleware, CorsMiddleware, AuthMiddleware)
            .forRoutes('*');
    }
}
```

### 7.5 全局中间件

```typescript
/**
 * 全局中间件 - main.ts
 * @author erik.zhou
 */

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { logger } from './common/middleware/logger.middleware';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);

    // 应用全局中间件
    app.use(logger);

    await app.listen(3000);
}
bootstrap();
```

---

## 第8章 管道与验证

### 8.1 内置管道

```typescript
/**
 * 内置管道
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    Post,
    Param,
    Body,
    Query,
    ParseIntPipe,
    ParseBoolPipe,
    ParseArrayPipe,
    ParseUUIDPipe,
    DefaultValuePipe,
} from '@nestjs/common';

@Controller('users')
export class UsersController {
    // ParseIntPipe - 转换为整数
    @Get(':id')
    findOne(@Param('id', ParseIntPipe) id: number) {
        return { userId: id, type: typeof id };
    }

    // ParseBoolPipe - 转换为布尔值
    @Get()
    findAll(@Query('active', ParseBoolPipe) active: boolean) {
        return { active: active, type: typeof active };
    }

    // ParseArrayPipe - 转换为数组
    @Get('by-ids')
    findByIds(
        @Query('ids', new ParseArrayPipe({ items: Number, separator: ',' }))
        ids: number[],
    ) {
        return { ids: ids };
    }

    // ParseUUIDPipe - 验证UUID
    @Get('uuid/:id')
    findByUUID(@Param('id', ParseUUIDPipe) id: string) {
        return { uuid: id };
    }

    // DefaultValuePipe - 设置默认值
    @Get('paginate')
    paginate(
        @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
        @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
    ) {
        return { page: page, limit: limit };
    }
}
```


### 8.2 ValidationPipe

```typescript
/**
 * ValidationPipe验证管道
 * @author erik.zhou
 */

// 安装依赖
// npm install class-validator class-transformer

import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);

    // 全局启用验证管道
    app.useGlobalPipes(
        new ValidationPipe({
            whitelist: true, // 自动删除非白名单属性
            forbidNonWhitelisted: true, // 存在非白名单属性时抛出错误
            transform: true, // 自动转换类型
            transformOptions: {
                enableImplicitConversion: true,
            },
        }),
    );

    await app.listen(3000);
}
bootstrap();
```

```typescript
/**
 * DTO验证
 * @author erik.zhou
 */

import {
    IsString,
    IsEmail,
    IsInt,
    Min,
    Max,
    IsOptional,
    MinLength,
    MaxLength,
    IsEnum,
    IsArray,
    ValidateNested,
    IsDate,
} from 'class-validator';
import { Type } from 'class-transformer';

enum UserRole {
    ADMIN = 'admin',
    USER = 'user',
}

class AddressDto {
    @IsString()
    city: string;

    @IsString()
    street: string;
}

export class CreateUserDto {
    @IsString()
    @MinLength(3)
    @MaxLength(20)
    name: string;

    @IsEmail()
    email: string;

    @IsInt()
    @Min(18)
    @Max(100)
    age: number;

    @IsString()
    @MinLength(8)
    password: string;

    @IsOptional()
    @IsString()
    phone?: string;

    @IsEnum(UserRole)
    role: UserRole;

    @IsArray()
    @IsString({ each: true })
    tags: string[];

    @ValidateNested()
    @Type(() => AddressDto)
    address: AddressDto;

    @IsDate()
    @Type(() => Date)
    birthDate: Date;
}
```

### 8.3 自定义管道

```typescript
/**
 * 自定义管道
 * @author erik.zhou
 */

import {
    PipeTransform,
    Injectable,
    ArgumentMetadata,
    BadRequestException,
} from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
    transform(value: string, metadata: ArgumentMetadata): number {
        const val = parseInt(value, 10);
        if (isNaN(val)) {
            throw new BadRequestException('验证失败：必须是数字');
        }
        return val;
    }
}
```

```typescript
/**
 * 自定义验证管道
 * @author erik.zhou
 */

import {
    PipeTransform,
    Injectable,
    ArgumentMetadata,
    BadRequestException,
} from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

@Injectable()
export class CustomValidationPipe implements PipeTransform<any> {
    async transform(value: any, { metatype }: ArgumentMetadata) {
        if (!metatype || !this.toValidate(metatype)) {
            return value;
        }

        const object = plainToClass(metatype, value);
        const errors = await validate(object);

        if (errors.length > 0) {
            const messages = errors.map(error => ({
                field: error.property,
                errors: Object.values(error.constraints || {}),
            }));

            throw new BadRequestException({
                message: '验证失败',
                errors: messages,
            });
        }

        return value;
    }

    private toValidate(metatype: Function): boolean {
        const types: Function[] = [String, Boolean, Number, Array, Object];
        return !types.includes(metatype);
    }
}
```

### 8.4 管道作用域

```typescript
/**
 * 管道作用域
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    Post,
    Body,
    Param,
    UsePipes,
    ParseIntPipe,
} from '@nestjs/common';
import { CustomValidationPipe } from './pipes/custom-validation.pipe';

@Controller('users')
export class UsersController {
    // 参数级别管道
    @Get(':id')
    findOne(@Param('id', ParseIntPipe) id: number) {
        return { userId: id };
    }

    // 方法级别管道
    @Post()
    @UsePipes(CustomValidationPipe)
    create(@Body() createUserDto: CreateUserDto) {
        return { message: '创建成功', user: createUserDto };
    }

    // 控制器级别管道
    @UsePipes(CustomValidationPipe)
    @Put(':id')
    update(
        @Param('id', ParseIntPipe) id: number,
        @Body() updateUserDto: UpdateUserDto,
    ) {
        return { message: '更新成功' };
    }
}
```

---

## 第9章 守卫与授权

### 9.1 基础守卫

```typescript
/**
 * 基础守卫
 * @author erik.zhou
 */

import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
    canActivate(
        context: ExecutionContext,
    ): boolean | Promise<boolean> | Observable<boolean> {
        const request = context.switchToHttp().getRequest();
        return this.validateRequest(request);
    }

    private validateRequest(request: any): boolean {
        const token = request.headers.authorization;
        return !!token;
    }
}
```

### 9.2 角色守卫

```typescript
/**
 * 角色守卫
 * @author erik.zhou
 */

import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
    constructor(private reflector: Reflector) {}

    canActivate(context: ExecutionContext): boolean {
        const requiredRoles = this.reflector.getAllAndOverride<string[]>(
            'roles',
            [context.getHandler(), context.getClass()],
        );

        if (!requiredRoles) {
            return true;
        }

        const request = context.switchToHttp().getRequest();
        const user = request.user;

        return requiredRoles.some(role => user?.roles?.includes(role));
    }
}
```

```typescript
/**
 * 使用角色守卫
 * @author erik.zhou
 */

import { Controller, Get, Post, UseGuards } from '@nestjs/common';
import { RolesGuard } from './guards/roles.guard';
import { Roles } from './decorators/roles.decorator';

@Controller('users')
@UseGuards(RolesGuard)
export class UsersController {
    @Get()
    @Roles('admin', 'user')
    findAll() {
        return { users: [] };
    }

    @Post()
    @Roles('admin')
    create() {
        return { message: '创建成功' };
    }

    @Delete(':id')
    @Roles('admin')
    remove() {
        return { message: '删除成功' };
    }
}
```

### 9.3 JWT守卫

```typescript
/**
 * JWT守卫
 * @author erik.zhou
 */

// 安装依赖
// npm install @nestjs/jwt @nestjs/passport passport passport-jwt

import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

```typescript
/**
 * JWT策略
 * @author erik.zhou
 */

import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
    constructor() {
        super({
            jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
            ignoreExpiration: false,
            secretOrKey: 'your-secret-key',
        });
    }

    async validate(payload: any) {
        return {
            userId: payload.sub,
            username: payload.username,
            roles: payload.roles,
        };
    }
}
```

```typescript
/**
 * 使用JWT守卫
 * @author erik.zhou
 */

import { Controller, Get, Post, UseGuards, Request } from '@nestjs/common';
import { JwtAuthGuard } from './guards/jwt-auth.guard';

@Controller('profile')
export class ProfileController {
    @Get()
    @UseGuards(JwtAuthGuard)
    getProfile(@Request() req) {
        return req.user;
    }
}
```

### 9.4 全局守卫

```typescript
/**
 * 全局守卫
 * @author erik.zhou
 */

import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { RolesGuard } from './guards/roles.guard';

@Module({
    providers: [
        {
            provide: APP_GUARD,
            useClass: RolesGuard,
        },
    ],
})
export class AppModule {}
```

---

## 第10章 拦截器

### 10.1 基础拦截器

```typescript
/**
 * 基础拦截器
 * @author erik.zhou
 */

import {
    Injectable,
    NestInterceptor,
    ExecutionContext,
    CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        const now = Date.now();
        const request = context.switchToHttp().getRequest();

        console.log(`请求开始: ${request.method} ${request.url}`);

        return next.handle().pipe(
            tap(() => {
                const duration = Date.now() - now;
                console.log(`请求结束: 耗时 ${duration}ms`);
            }),
        );
    }
}
```


### 10.2 响应转换拦截器

```typescript
/**
 * 响应转换拦截器
 * @author erik.zhou
 */

import {
    Injectable,
    NestInterceptor,
    ExecutionContext,
    CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
    code: number;
    message: string;
    data: T;
    timestamp: number;
}

@Injectable()
export class TransformInterceptor<T>
    implements NestInterceptor<T, Response<T>>
{
    intercept(
        context: ExecutionContext,
        next: CallHandler,
    ): Observable<Response<T>> {
        return next.handle().pipe(
            map(data => ({
                code: 0,
                message: '成功',
                data: data,
                timestamp: Date.now(),
            })),
        );
    }
}
```

### 10.3 异常拦截器

```typescript
/**
 * 异常拦截器
 * @author erik.zhou
 */

import {
    Injectable,
    NestInterceptor,
    ExecutionContext,
    CallHandler,
    HttpException,
    HttpStatus,
} from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor implements NestInterceptor {
    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        return next.handle().pipe(
            catchError(err => {
                if (err instanceof HttpException) {
                    return throwError(() => err);
                }

                return throwError(
                    () =>
                        new HttpException(
                            {
                                code: 1,
                                message: '服务器内部错误',
                                error: err.message,
                            },
                            HttpStatus.INTERNAL_SERVER_ERROR,
                        ),
                );
            }),
        );
    }
}
```

### 10.4 缓存拦截器

```typescript
/**
 * 缓存拦截器
 * @author erik.zhou
 */

import {
    Injectable,
    NestInterceptor,
    ExecutionContext,
    CallHandler,
} from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
    private cache = new Map();

    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        const request = context.switchToHttp().getRequest();
        const key = `${request.method}:${request.url}`;

        // 检查缓存
        if (this.cache.has(key)) {
            console.log('从缓存返回');
            return of(this.cache.get(key));
        }

        // 执行请求并缓存结果
        return next.handle().pipe(
            tap(response => {
                console.log('缓存响应');
                this.cache.set(key, response);

                // 5分钟后清除缓存
                setTimeout(() => {
                    this.cache.delete(key);
                }, 300000);
            }),
        );
    }
}
```

### 10.5 超时拦截器

```typescript
/**
 * 超时拦截器
 * @author erik.zhou
 */

import {
    Injectable,
    NestInterceptor,
    ExecutionContext,
    CallHandler,
    RequestTimeoutException,
} from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        return next.handle().pipe(
            timeout(5000), // 5秒超时
            catchError(err => {
                if (err instanceof TimeoutError) {
                    return throwError(
                        () => new RequestTimeoutException('请求超时'),
                    );
                }
                return throwError(() => err);
            }),
        );
    }
}
```

### 10.6 使用拦截器

```typescript
/**
 * 使用拦截器
 * @author erik.zhou
 */

import {
    Controller,
    Get,
    UseInterceptors,
} from '@nestjs/common';
import { LoggingInterceptor } from './interceptors/logging.interceptor';
import { TransformInterceptor } from './interceptors/transform.interceptor';

@Controller('users')
export class UsersController {
    // 方法级别拦截器
    @Get()
    @UseInterceptors(LoggingInterceptor)
    findAll() {
        return { users: [] };
    }

    // 多个拦截器
    @Get(':id')
    @UseInterceptors(LoggingInterceptor, TransformInterceptor)
    findOne() {
        return { user: { id: 1, name: 'John' } };
    }
}
```

```typescript
/**
 * 全局拦截器 - main.ts
 * @author erik.zhou
 */

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { TransformInterceptor } from './interceptors/transform.interceptor';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);

    // 全局拦截器
    app.useGlobalInterceptors(new TransformInterceptor());

    await app.listen(3000);
}
bootstrap();
```

---

## 总结

本教程全面介绍了NestJS框架的核心概念和实战应用：

1. **基础知识**：项目结构、配置、生命周期
2. **装饰器系统**：类装饰器、方法装饰器、参数装饰器、自定义装饰器
3. **依赖注入**：基础注入、作用域、自定义提供者、循环依赖
4. **模块系统**：功能模块、共享模块、动态模块、模块重导出
5. **控制器**：路由处理、DTO、响应处理、异常处理、版本控制
6. **服务提供者**：基础服务、服务依赖、异步服务
7. **中间件**：函数式中间件、类中间件、全局中间件
8. **管道与验证**：内置管道、ValidationPipe、自定义管道
9. **守卫与授权**：基础守卫、角色守卫、JWT守卫
10. **拦截器**：日志拦截器、响应转换、异常处理、缓存、超时

### 最佳实践

1. **模块化设计**：按功能划分模块，保持模块职责单一
2. **依赖注入**：充分利用DI系统，提高代码可测试性
3. **DTO验证**：使用class-validator进行数据验证
4. **异常处理**：统一异常处理，提供友好的错误信息
5. **中间件顺序**：合理安排中间件执行顺序
6. **守卫使用**：实现细粒度的权限控制
7. **拦截器应用**：统一响应格式，实现横切关注点
8. **代码规范**：遵循TypeScript和NestJS最佳实践

### 进阶学习

1. **数据库集成**：TypeORM、Prisma、Mongoose
2. **微服务**：gRPC、消息队列、服务发现
3. **GraphQL**：Apollo Server集成
4. **WebSocket**：实时通信
5. **任务调度**：定时任务、队列处理
6. **测试**：单元测试、集成测试、E2E测试
7. **部署**：Docker容器化、CI/CD
8. **性能优化**：缓存策略、数据库优化

### 推荐资源

- [NestJS官方文档](https://docs.nestjs.com/)
- [NestJS中文文档](https://docs.nestjs.cn/)
- [TypeScript官方文档](https://www.typescriptlang.org/)
- [NestJS GitHub](https://github.com/nestjs/nest)

---

**@author erik.zhou**
**最后更新时间：** 2026-03-02

