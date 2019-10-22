# 模块
要使用 CLI 创建模块，只需执行 $ nest g module cats 命令。

## 功能模块
CatsController 和 CatsService 属于同一个应用程序域。 应该考虑将它们移动到一个功能模块下，即 CatsModule。

#### cats/cats.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}

现在已经创建了 cats.module.ts 文件，并把与这个模块相关的所有东西都移到了 cats 目录下。
我们需要做的最后一件事是将这个模块导入根模块 (ApplicationModule)。

#### app.module.ts

import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class ApplicationModule {}

现在 Nest 知道除了 ApplicationModule 之外，注册 CatsModule 也是非常重要的。 这就是我们现在的目录结构:

src
├──cats
│    ├──dto
│    │   └──create-cat.dto.ts
│    ├──interfaces
│    │     └──cat.interface.ts
│    ├─cats.service.ts
│    ├─cats.controller.ts
│    └──cats.module.ts
├──app.module.ts
└──main.ts

## 全局模块
Nest 将提供者封装在模块范围内。您无法在其他地方使用模块的提供者而不导入他们。
但是有时候，你可能只想提供一组随时可用的东西 - 例如：helper，数据库连接等等。这时候就可以使模块成为全局模块。

import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}

@Global 装饰器使模块成为全局作用域。 全局模块应该只注册一次，最好由根或核心模块注册。 
之后，CatsService 组件将无处不在，但 CatsModule 不会被导入。

使一切全局化并不是一个好的解决方案。 全局模块可用于减少必要模板文件的数量。 imports 数组仍然是使模块 API 透明的最佳方式。

## 动态模块
Nest 模块系统带有一个称为动态模块的功能。 它使您能够毫不费力地创建可定制的模块。 让我们来看看 DatabaseModule：

import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}

forRoot() 可以同步或异步（Promise）返回动态模块。

此模块默认定义了 Connection 提供者，但另外 - 根据传递的 options 和 entities - 创建一个提供者集合，例如存储库。
实际上，动态模块扩展了模块元数据。当您需要动态注册组件时，这个实质特性非常有用。然后你可以通过以下方式导入 DatabaseModule：

import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class ApplicationModule {}

为了导出动态模块，可以省略函数调用部分：

import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class ApplicationModule {}
