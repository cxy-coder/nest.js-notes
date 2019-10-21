# nest.js-notes
## 控制器（{name}.controller.ts）
### 路由
使用CLI创建控制器： $ nest g controller {name}

import { Controller, Get } from '@nestjs/common';

@Controller({name})
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
可以在@Controller()、@Get()装饰器中为其添加路由前缀；

### Request对象

import { Request } from 'express';
import { Controller, Get } from '@nestjs/common';

@Controller({name})
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}

通过@Req()装饰器将请求中的对象或数据注入处理程序；

Request 对象表示 HTTP 请求，并具有 Request 查询字符串，参数，HTTP 标头 和 正文的属性，但在大多数情况下, 不必手动获取它们。 
我们可以使用专用的装饰器，比如开箱即用的 @Body() 或 @Query() 。 下面是装饰器和普通表达对象的比较。

@Request()	req
@Response()	res
@Next()	next
@Session()	req.session
@Param(key?: string)	req.params / req.params[key]
@Body(key?: string)	req.body / req.body[key]
@Query(key?: string)	req.query / req.query[key]
@Headers(name?: string)	req.headers / req.headers[name]

Nest提供了@Res()/@Response()装饰器，负责管理响应，调用响应对象--res.json(...)或res.send(...)发出某种响应，否则HTTP服务器将挂起；

Nest以相同的方式提供其余的端点装饰器- @Put() 、 @Delete()、 @Patch()、 @Options()、 @Head()和 @All()。这些表示各自的 HTTP请求方法。

### 路由通配符
路由同样支持模式匹配。例如，星号被用作通配符，将匹配任何字符组合。

@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
Copy to clipboardErrorCopied
以上路由地址将匹配 abcd 、 ab_cd 、 abecd 等。字符 ? 、 + 、 * 以及 () 是它们的正则表达式对应项的子集。连字符 (-) 和点 (.) 按字符串路径解析。

### 状态码
如前面所说，默认情况下，响应的状态码总是200，除了 POST 请求外，它是201，我们可以通过在处理程序层添加@HttpCode（...） 装饰器来更改状态码。

@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
Copy to clipboardErrorCopied
 
 ▶️HttpCode 需要从 @nestjs/common 包导入。

通常状态码不是固定的，您可以使用类库特有的的响应（通过@Res()注入 ）对象（或者，在出现错误时，抛出异常）。

### Headers
使用 @header() 修饰器或类库特有的响应对象要指定自定义响应头,（使用 res.header()直接调用）。

@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
Copy to clipboardErrorCopied

▶️Header 需要从 @nestjs/common 包导入。

### Redirection (重定向)
要将响应重定向到特定的 URL，可以使用 @Redirect()装饰器或特定于库的响应对象(调用： res.redirect())。

@Redirect() 带有必需的 url参数和可选的 statusCode参数。 如果省略，则 statusCode 默认为 302。

@Get()
@Redirect('https://nestjs.com', 301)

要动态确定HTTP状态代码或重定向URL，可通过从路由处理程序方法返回一个形状为以下形式的对象：
{
  "url": string,
  "statusCode": number
}

返回的值将覆盖传递给 @Redirect()装饰器的所有参数。 例如：

@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}

### 路由参数
当您需要接受动态数据作为请求的一部分时（例如，使用GET /cats/1来获取 id为 1的 cat），我们可以在路由中添加路由参数标记，以捕获请求 URL 中该位置的动态值。
可以使用 @Param() 装饰器访问以这种方式声明的路由参数，该装饰器应添加到函数签名中。

@Get(':id')
findOne(@Param() params): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}

@Param()用于修饰方法参数，并使路由参数可用作该修饰的方法参数在方法体内的属性。我们可以通过名称直接引用路由参数--params.id来访问 id参数。

▶️Param 需要从 @nestjs/common 包导入。

@Get(':id')
findOne(@Param('id') id): string {
  return `This action returns a #${id} cat`;
}

### 请求负载
首先我们需要确定 DTO(数据传输对象)模式。DTO是一个对象。

我们来创建 CreateCatDto 类：

create-cat.dto.ts

export class CreateCatDto {
  readonly name: string;
  readonly age: number;
  readonly breed: string;
}

它只有三个基本属性。 之后，我们可以在 CatsController中使用新创建的DTO：

cats.controller.ts

@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}

### 完整示例
cats.controller.ts

import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}

### 最后一步
控制器已经准备就绪，可以使用，但是 Nest 不知道 CatsController 是否存在，所以它不会创建这个类的一个实例。

控制器总是属于模块，这就是为什么我们将 controllers 数组保存在 @module() 装饰器中。 

app.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}

使用 @Module()装饰器将元数据附加到模块类。
