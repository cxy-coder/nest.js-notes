# 提供者
要使用 CLI 创建服务类，只需执行 $ nest g service cats 命令

## 服务
服务负责数据存储和检索，由 CatsController 其使用。因此，用 @Injectable() 这个类来装饰。

cats.service.ts

import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}

## 可选的提供者
有时，您可能需要解决一些依赖项。例如，您的类可能依赖于一个配置对象，但如果没有传递，则应使用默认值。
在这种情况下，关联变为可选的，提供者不会因为缺少配置导致错误。

要指示提供程序是可选的，请在 constructor 的参数中使用 @optional() 装饰器。

import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(
    @Optional() @Inject('HTTP_OPTIONS') private readonly httpClient: T
  ) {}
}

请注意，在上面的示例中，我们使用自定义提供程序，这是我们包含 HTTP_OPTIONS自定义标记的原因。
前面的示例显示了基于构造函数的注入，通过构造函数中的类指示依赖关系。在此处详细了解自定义提供程序及其关联的 token。

## 注册提供者
现在我们已经定义了提供程序（CatsService），并且已经有了该服务的使用者（CatsController），我们需要在 Nest中注册该服务，以便它可以执行注入。 
为此，我们可以编辑模块文件（app.module.ts），然后将服务添加到@Module()装饰器的providers数组中。

app.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}

得益于此，Nest 现在将能够解决 CatsController 类的依赖关系。这就是我们目前的目录结构：

src
├── cats
│    ├──dto
│    │   └──create-cat.dto.ts
│    ├── interfaces
│    │       └──cat.interface.ts
│    ├──cats.service.ts
│    └──cats.controller.ts
├──app.module.ts
└──main.ts



