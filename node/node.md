## 1.使用NodeJS编写代码实现遍历文件夹及所有文件名

```javascript
const fs=require('fs')
const path = require('path')

const readDir = (entry) => {
  const dirInfo = fs.readdirSync(entry)
  dirInfo.forEach(item => {
    const location = path.join(entry, item)
    const info=fs.statSync(location)
    if (info.isDirectory()) {
      console.log(`dir:${location}`)
      readDir(location)
    } else {
      console.log(`file:${location}`)
      
    }
  })
}

readDir(__dirname)
```

## 2. node如何做版本的升级？为什么要使用nvm？

有新api时，提高webpack的打包速度

nvm允许在你电脑上安装不同版本的node，nvm管理node版本

```
nvm ls
nvm install v11.0.3
nvm use v11.0.3
```

## 3.模块化的差异，AMD,CMD,COMMONJS,ESMODULE

https://juejin.im/post/5aaa37c8f265da23945f365c

1. AMD：依赖前置，立即执行

   ```
   define(['a','b'],function(a,b){
   
   })
   ```

   异步，模块不偶和，动态引入

   **AMD是RequireJS**在推广过程中**对模块定义的规范化**产出，它是一个概念，RequireJS是对这个概念的实现，就好比JavaScript语言是对ECMAScript规范的实现。AMD是一个组织，RequireJS是在这个组织下自定义的一套脚本语言

   RequireJS：是一个AMD框架，可以异步加载JS文件，按照模块加载方法，通过define()函数定义，第一个参数是一个数组，里面定义一些需要依赖的包，第二个参数是一个回调函数，通过变量来引用模块里面的方法，最后通过return来输出。

   是一个依赖前置、异步定义的AMD框架（在参数里面引入js文件），在定义的同时如果需要用到别的模块，在最前面定义好即在参数数组里面进行引入，在回调里面加载

2. CMD：依赖就近，延迟执行

   **CMD---**是**SeaJS**在推广过程中对模块定义的规范化产出，是一个同步模块定义，是SeaJS的一个标准，SeaJS是CMD概念的一个实现，SeaJS是淘宝团队提供的一个模块开发的js框架.

   通过define()定义，没有依赖前置，通过require加载jQuery插件，CMD是依赖就近，在什么地方使用到插件就在什么地方require该插件，即用即返，这是一个同步的概念

3. COMMONJS

   ```
   require()
   require()
   ```

   同步,有一点耦合,引入可以动态引入

   **CommonJS规范--**-是通过**module.exports定**义的，在前端浏览器里面并不支持module.exports,通过node.js后端使用的。Nodejs端是使用CommonJS规范的，前端浏览器一般使用AMD、CMD、ES6等定义模块化开发的

4. ESMODULE

   静态引入

   **ES6特性，模块化**---**export/import对模块进行导出导入的**

## 4.图片上传到服务器的过程(FileReader.readAsDataURL)

```javascript
<input type='file' onchange=function(){e}>
<img src='base64:sadafasfasfasf' />
```

1. 高版本浏览器用`FileReader的类`

2. 全浏览器如何兼容

   没有FileReader

   // onchange：提交给后端，后端存储完返回一个图片的url，再放到img src里面

## 5.token存在cookie里，过期怎么处理

- ## 业务场景

  在前后分离场景下，越来越多的项目使用token作为接口的安全机制，APP端或者WEB端（使用VUE、REACTJS等构建）使用token与后端接口交互，以达到安全的目的。本文结合stackover以及本身项目实践，试图总结出一个通用的，可落地的方案。

  ## 基本思路

  - 单个token

  1. token（A）过期设置为15分钟
  2. 前端发起请求，后端验证token（A）是否过期；如果过期，前端发起刷新token请求，后端设置已再次授权标记为true，请求成功
  3. 前端发起请求，后端验证再次授权标记，如果已经再次授权，则拒绝刷新token的请求，请求成功
  4. 如果前端每隔72小时，必须重新登录，后端检查用户最后一次登录日期，如超过72小时，则拒绝刷新token的请求，请求失败

  - 授权token加上刷新token

  用户仅登录一次，用户改变密码，则废除token，重新登录

  ## 1.0实现

  1.登录成功，返回access_token和refresh_token，客户端缓存此两种token;
  2.使用access_token请求接口资源，成功则调用成功；如果token超时，客户端
  携带refresh_token调用中间件接口获取新的access_token;
  3.中间件接受刷新token的请求后，检查refresh_token是否过期。
  如过期，拒绝刷新，客户端收到该状态后，跳转到登录页；
  如未过期，生成新的access_token和refresh_token并返回给客户端（如有可能，让旧的refresh_token失效），客户端携带新的access_token重新调用上面的资源接口。
  4.客户端退出登录或修改密码后，调用中间件注销旧的token(使access_token和refresh_token失效)，同时清空客户端的access_token和refresh_toke。

  后端表
  id user_id client_id client_secret refresh_token expire_in create_date del_flag

  ## 2.0实现

  场景： access_token访问资源 refresh_token授权访问 设置固定时间X必须重新登录

  1.登录成功，后台jwt生成access_token（jwt有效期30分钟）和refresh_token（jwt有效期15天），并缓存到redis（hash-key为token,sub-key为手机号,value为设备唯一编号（根据手机号码，可以人工废除全部token，也可以根据sub-key,废除部分设备的token。），设置过期时间为1个月，保证最终所有token都能删除)，返回后，客户端缓存此两种token;
  2.使用access_token请求接口资源，校验成功且redis中存在该access_token（未废除）则调用成功；如果token超时，中间件删除access_token（废除）；客户端再次携带refresh_token调用中间件接口获取新的access_token;
  3.中间件接受刷新token的请求后，检查refresh_token是否过期。
  如过期，拒绝刷新，删除refresh_token（废除）； 客户端收到该状态后，跳转到登录页；
  如未过期，检查缓存中是否有refresh_token（是否被废除），如果有，则生成新的access_token并返回给客户端，客户端接着携带新的access_token重新调用上面的资源接口。
  4.客户端退出登录或修改密码后，调用中间件注销旧的token(中间件删除access_token和refresh_token（废除）)，同时清空客户端侧的access_token和refresh_toke。
  5.如手机丢失，可以根据手机号人工废除指定用户设备关联的token。
  6.以上3刷新access_token可以增加根据登录时间判断最长X时间必须重新登录，此时则拒绝刷新token。（拒绝的场景：失效，长时间未登录，频繁刷新）

  2.0 变动
  1.登录
  2.登录拦截器
  3.增加刷新access_token接口
  4.退出登录
  5.修改密码

  ## 3.0实现

  场景：自动续期 长时间未使用需重新登录

  1.登录成功，后台jwt生成access_token（jwt有效期30分钟），并缓存到redis（hash-key为access_token,sub-key为手机号,value为设备唯一编号（根据手机号码，可以人工废除全部token），设置access_token过期时间为7天，保证最终所有token都能删除)，返回后，客户端缓存此token;

  2.使用access_token请求接口资源，校验成功且redis中存在该access_token（未废除）则调用成功；如果token超时，中间件删除access_token（废除）,同时生成新的access_token并返回。客户端收到新的access_token，
  再次请求接口资源。

  3.客户端退出登录或修改密码后，调用中间件注销旧的token(中间件删除access_token（废除）)，同时清空客户端侧的access_token。

  4.以上2 可以增加根据登录时间判断最长X时间必须重新登录，此时则拒绝刷新token。（拒绝的场景：长时间未登录，频繁刷新）

  5.如手机丢失，可以根据手机号人工废除指定用户设备关联的token。

  3.0 变动

  1.登录
  2.登录拦截器
  3.退出登录
  4.修改密码

  1.3 场景：token过期重新登录 长时间未使用需重新登录

  1.登录成功，后台jwt生成access_token（jwt有效期7天），并缓存到redis，key为 "user_id:access_token",value为access_token（根据用户id，可以人工废除指定用户全部token），设置缓存过期时间为7天，保证最终所有token都能删除，请求返回后，客户端缓存此access_token；

  2.使用access_token请求接口资源，校验成功且redis中存在该access_token（未废除）则调用成功；如果token超时，中间件删除access_token（废除）,同时生成新的access_token并返回。客户端收到新的access_token，
  再次请求接口资源。

  3.客户端退出登录或修改密码后，调用中间件注销旧的token(中间件删除access_token（废除）)，同时清空客户端侧的access_token。

  4.以上2 可以增加根据登录时间判断最长X时间必须重新登录，此时则拒绝刷新token。（拒绝的场景：长时间未登录，频繁刷新）

  5.如手机丢失，可以根据手机号人工废除指定用户设备关联的token。

  1.3 变动

  1.登录
  2.登录拦截器
  3.退出登录
  4.修改密码

  # 解决方案

  2.0 场景： access_token访问资源 refresh_token授权访问 设置固定时间X必须重新登录

  1.登录成功，后台jwt生成access_token（jwt有效期30分钟）和refresh_token（jwt有效期15天），并缓

  存到redis（hash-key为token,sub-key为手机号,value为设备唯一编号（根据手机号码，可以人工废除全

  部token，也可以根据sub-key,废除部分设备的token。），设置过期时间为1个月，保证最终所有token都

  能删除)，返回后，客户端缓存此两种token;

  2.使用access_token请求接口资源，校验成功且redis中存在该access_token（未废除）则调用成功；如果

  token超时，中间件删除access_token（废除）；客户端再次携带refresh_token调用中间件接口获取新的

  access_token;

  3.中间件接受刷新token的请求后，检查refresh_token是否过期。

  如过期，拒绝刷新，删除refresh_token（废除）； 客户端收到该状态后，跳转到登录页；

  如未过期，检查缓存中是否有refresh_token（是否被废除），如果有，则生成新的access_token并返回给

  客户端，客户端接着携带新的access_token重新调用上面的资源接口。

  4.客户端退出登录或修改密码后，调用中间件注销旧的token(中间件删除access_token和refresh_token（

  废除）)，同时清空客户端侧的access_token和refresh_toke。

  5.如手机丢失，可以根据手机号人工废除指定用户设备关联的token。

  6.以上3刷新access_token可以增加根据登录时间判断最长X时间必须重新登录，此时则拒绝刷新token。（

  拒绝的场景：失效，长时间未登录，频繁刷新）

  2.0 变动

  1.登录

  2.登录拦截器

  3.增加刷新access_token接口

  4.退出登录

  5.修改密码

  3.0 场景：自动续期 长时间未使用需重新登录

  1.登录成功，后台jwt生成access_token（jwt有效期30分钟），并缓存到redis（hash-key为

  access_token,sub-key为手机号,value为设备唯一编号（根据手机号码，可以人工废除全部token，也可以

  根据sub-key,废除部分设备的token。），设置access_token过期时间为1个月，保证最终所有token都能删

  除)，返回后，客户端缓存此token;

  2.使用access_token请求接口资源，校验成功且redis中存在该access_token（未废除）则调用成功；如果

  token超时，中间件删除access_token（废除）,同时生成新的access_token并返回。客户端收到新的

  access_token，

  再次请求接口资源。

  3.客户端退出登录或修改密码后，调用中间件注销旧的token(中间件删除access_token（废除）)，同时清

  空客户端侧的access_token。

  4.以上2 可以增加根据登录时间判断最长X时间必须重新登录，此时则拒绝刷新token。（拒绝的场景：长

  时间未登录，频繁刷新）

  5.如手机丢失，可以根据手机号人工废除指定用户设备关联的token。

  3.0 变动

  1.登录

  2.登录拦截器

  3.退出登录

  4.修改密码

  4.0 场景：token过期重新登录 长时间未使用需重新登录

  1.登录成功，后台jwt生成access_token（jwt有效期7天），并缓存到redis，key为

  "user_id:access_token" + 用户id,value为access_token（根据用户id，可以人工废除指定用户全部

  token），设置缓存过期时间为7天，保证最终所有token都能删除，请求返回后，客户端缓存此

  access_token；

  2.使用access_token请求接口资源，校验成功且redis中存在该access_token（未废除）则调用成功；如果

  token超时，中间件删除access_token（废除）,同时生成新的access_token并返回。客户端收到新的

  access_token，

  再次请求接口资源。

  3.客户端退出登录或修改密码后，调用中间件注销旧的token(中间件删除access_token（废除）)，同时清

  空客户端侧的access_token。

  4.以上2 可以增加根据登录时间判断最长X时间必须重新登录，此时则拒绝刷新token。（拒绝的场景：长

  时间未登录，频繁刷新）

  5.如手机丢失，可以根据手机号人工废除指定用户设备关联的token。

  4.0 变动

  1.登录

  2.登录拦截器

  3.退出登录

  4.修改密码

  ## 最终实现

  ### 后端

  1. 在登录接口中 如果校验账号密码成功 则根据用户id和用户类型创建jwt token(有效期设置为-1，即永不过期),得到A
  2. 更新登录日期(当前时间new Date()即可)（业务上可选），得到B
  3. 在redis中缓存key为ACCESS_TOKEN:userId:A(加上A是为了防止用户多个客户端登录 造成token覆盖),value为B的毫秒数（转换成字符串类型），过期时间为7天（7 * 24 * 60 * 60）
  4. 在登录结果中返回json格式为{"result":"success","token": A}
  5. 用户在接口请求header中携带token进行登录，后端在所有接口前置拦截器进行拦截，作用是解析token 拿到userId和用户类型（用户调用业务接口只需要传token即可）， 如果解析失败（抛出SignatureException），则返回json（code = 0 ,info= Token验证不通过, errorCode = '1001'）； 此外如果解析成功，验证redis中key为ACCESS_TOKEN:userId:A 是否存在 如果不存在 则返回json（code = 0 ,info= 会话过期请重新登录, errorCode = '1002'）； 如果缓存key存在，则自动续7天超时时间（value不变），实现频繁登录用户免登陆。
  6. 把userId和用户类型放入request参数中 接口方法中可以直接拿到登录用户信息
  7. 如果是修改密码或退出登录 则废除access_tokens（删除key）

  ### 前端（VUE）

  1. 用户登录成功，则把username存入cookie中，key为loginUser;把token存入cookie中，key为accessToken 把token存入Vuex全局状态中
  2. 进入首页


## 6.node框架的mvc

**Model（模型）**是应用程序中用于处理应用程序数据逻辑的部分。
　　通常模型对象负责在数据库中存取数据。

**View（视图）**是应用程序中处理数据显示的部分。
　　通常视图是依据模型数据创建的。

**Controller（控制器）**是应用程序中处理用户交互的部分。
　　通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

## 7.mongodb和mysql的优势

##### MongoDB优点：

- 性能优越：快速！在适量级的内存的MongoDB的性能是非常迅速的，它将热数据存储在物理内存中，使得热数据的读写变得十分快。
- 高扩展：第三方支持丰富（这是与其他的No SQL相比，MongoDB也具有的优势）
- 自身的Failover机制！
- 弱一致性（最终一致），更能保证用户的访问速度
- 文档结构的存储方式，能够更便捷的获取数据：json的存储格式
- 支持大容量的存储，内置GridFS
- 内置Sharding

##### MongoDB 缺点：

主要是无事务机制
 ① MongoDB 不支持事务操作(最主要的缺点)
 ② MongoDB 占用空间过大
 ③ MongoDB 没有如 MySQL 那样成熟的维护工具，这对于开发和IT运营都是个值得注意的地方

##### Redis：

它就是一个不折不扣的内存数据库。
 持久化方式：
 Redis 所有数据都是放在内存中的，持久化是使用 RDB 方式或者 aof 方式。

##### MySQL：

无论数据还是索引都存放在硬盘中。到要使用的时候才交换到内存中。能够处理远超过内存总量的数据。
 关系型数据库。
 在不同的引擎上有不同 的存储方式。
 查询语句是使用传统的 SQL 语句，拥有较为成熟的体系，成熟度很高。
 开源数据库的份额在不断增加，MySQL 的份额页在持续增长。

缺点就是在海量数据处理的时候效率会显著变慢。

## 8.less(js),sass(ruby),stylus,css,命名空间与css module

命名空间

```scss
.home{
   .home-header{}
   .home-content{}
}
```

可以设置css module

## 9.工程化上的按需加载

按需加载，配置webpack，require.ensure();import()

## 10.git上的冲突怎么解决

多人改统一文件时出现冲突，需要手动协调

## 11.设计模式

发布订阅，观察者，组合，继承，单例模式

## 12.node中npm与版本管理（package.lock，yarn.lock）

锁住版本

## 13.webpack

## 14.后端环境的搭建

pm2

## 15.typescript

