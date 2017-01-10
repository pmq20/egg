title: Service
---

# Service
`Service` 简单来说，就是当遇到一个复杂的业务场景，我们会把这些逻辑封装到一起，作为一个 `Service`，这样可以保持 Controller 的简单，而且也可以保证业务逻辑的独立性， 便于后面的重构和代码重用。

## Service 的一些场景

- 复杂数据的处理，比如要展现的信息需要从数据库获取，还要经过一定的规则计算，才能返回用户显示。或者计算完成后，更新到数据库。
- 第三方服务的调用，比如 Github 信息获取等。

## 定义 Service

- `app/services/user.js`

  ```js
  module.exports = app => {
    class User extends app.Service {
      constructor(ctx) {
        super(ctx);
      }
      * find(uid) {
      }
    }
    return User;
  };
  ```

### 注意事项
- Service 文件必须放在 `app/services` 目录，可以支持多级目录，访问的时候可以通过目录名级联访问。

  ```
  app/services/biz/user.js => this.services.biz.user.find
  ```

- 一个 Service 文件只能包含一个类， 这个类需要通过 `module.exports` 的方式返回。
- Service 需要通过 Class 的方式定义，父类必须是 app.Service, 其中 app.Service 会在初始化 Service 的时候，传递进来。

## 使用 Service

下面就通过一个完整的例子，看看怎么使用 Service。

```js
// app/router.js
module.exports = app => {
  app.get('/user/:id', 'user.info');
};

// app/controller/user.js
exports.info = function* () {
  const userId = this.params.id;
  const userInfo = yield this.services.user.find(userId);
  this.body = userInfo;
};

// app/services/user.js
module.exports = app => {
  class User extends app.Service {
    constructor(ctx) {
      super(ctx);
    }

    * find(uid) {
      // 假如 我们拿到用户 id 从数据库获取用户详细信息
      const user = this.db.query(`select * from user where uid = ${uid}`);
      // 假定这里还有一些复杂的计算，然后返回需要的信息。
      return {
        name: user.user_name,
        age: user.age,
      };
    }
  }
  return User;
};

// curl http://127.0.0.1:7001/user/1234
```
