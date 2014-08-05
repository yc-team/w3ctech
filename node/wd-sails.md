sails
================

#### 安装

```shell
sudo npm install sails -g
```

#### 创建并开启服务

```shell
sails new wd-w3ctech
cd wd-w3ctech
sails lift
```

#### 创建Controller

```shell
sails generate controller user index show edit delete
```

注释：不能创建2次一样的

> error: Something else already exists at ::/Users/***/sails/wd-w3ctech/api/controllers/UserController.js











#### .sailsrc文件

```
{
  "generators": {
    "modules": {}
  }
}
```



#### policies

在Sails里面，policies对于授权和访问控制都是很好的工具：可以很好地同意和拒绝访问你的controlls。

举例：比如我们做一个Dropbox，在让一个用户上传一个文件到一个目录的时候，你会去检查她是不是授权的，然后再确认她是不是有写的权限，最后，你会检查她上传的这个目录有没有足够的空间。

policies可以做任何如下的：

* HTTP BasicAuth
* 3rd party single-sign-on
* OAuth 2.0
或者你自己定义的custom authorization/authentication scheme.

> 注意：policies只是应用于controller actions，而不是views。如果你在routes.js config文件里面定义了一个route，那个直接应用于view，而不是policies。你可以定义一个controll action - 它用来展示你的view，然后控制你的route指向那个action。

policies都是定义在你Sails app里面的api/policies文件夹的文件。

每一个policy文件应该只包含一个function。

下面这个例子，policies都只是在你的controllers前面执行的Connect/Express middleware的方法。理想情况下，每一个middleware方法应该只检测一个事情。

下面的canWrite policy大概写出这样：

```
// policies/canWrite.js
module.exports = function canWrite (req, res, next) {
  var targetFolderId = req.param('id');
  var userId = req.session.user.id;

  Permission
  .findOneByFolderId( targetFolderId )
  .exec( function foundPermission (err, permission) {

    // Unexpected error occurred-- skip to the app's default error (500) handler
    if (err) return next(err);

    // No permission exists linking this user to this folder.  Maybe they got removed from it?  Maybe they never had permission in the first place?  Who cares?
    if ( ! permission ) return res.redirect('/notAllowed');

    // OK, so a permission was found.  Let's be sure it's a "write".
    if ( permission.type !== 'write' ) return res.redirect('/notAllowed');

    // 到了这里了，看起来都ok了，让用户过把
    next();
  });
};
```


* 用policies来保护controllers

sails在config/policies.js里面有一个ACL (access control list) 的配置。

这个文件用于配置你的controllers的policies。

你的config/policies.js文件应该输出一个js对象：

1、包含key - controll的名称或者是 * （代表全局的policies）
2、包含values - 是一个对象：key是action的名字，值是policies

下面有一个例子：

```
{
  ProfileController: {
      // Apply the 'isLoggedIn' policy to the 'edit' action of 'ProfileController'
      edit: 'isLoggedIn'
      // Apply the 'isAdmin' AND 'isLoggedIn' policies, in that order, to the 'create' action
      create: ['isAdmin', 'isLoggedIn']
  }
}
```

将一个policy应用于一整个controller：


```
{
  ProfileController: {
    // Apply 'isLogged' in by default to all actions that are NOT specified below
    '*': 'isLoggedIn',
    // If an action is explicitly listed, its policy list will override the default list.
    // So, we have to list 'isLoggedIn' again for the 'edit' action if we want it to be applied.
    edit: ['isAdmin', 'isLoggedIn']
  }
}
```

> 默认的policy不会cascade 或者 trickle down. 指定的controller's actions会覆盖默认的，如上面例子，如果你还想给edit这个action加上isLoggedIn的policy，就需要再写一次。



将一个policy应用于所有的没有被明确定义的actions

```
{
  // Apply 'isLoggedIn' to all actions by default
  '*': 'isLoggedIn',
  ProfileController: {
      // Apply 'isAdmin' to the 'foo' action.  'isLoggedIn' will NOT be applied!
      'foo': 'isAdmin'
  }
}
```

> 注意：默认的policies不会应用于那些被指定明确policy的controller / action。



* 内置的policies

sails支持两种内置的policies，可以应用于全局或者特定的controller或者action。

* true
* false
* '*': true

给一个controll添加一些policies：

```
 // in config/policies.js

  // ...
  RabbitController: {

    // Apply the `false` policy as the default for all of RabbitController's actions
    // (`false` prevents all access, which ensures that nothing bad happens to our rabbits)
    '*': false,

    // For the action `nurture`, apply the 'isRabbitMother' policy 
    // (this overrides `false` above)
    nurture : 'isRabbitMother',

    // Apply the `isNiceToAnimals` AND `hasRabbitFood` policies
    // before letting any users feed our rabbits
    feed : ['isNiceToAnimals', 'hasRabbitFood']
  }
  // ...
```


policies/isNiceToAnimals.js下的isNiceToAnimals policy：

```
module.exports = function isNiceToAnimals (req, res, next) {

  // `req.session` contains a set of data specific to the user making this request.
  // It's kind of like our app's "memory" of the current user.

  // If our user has a history of animal cruelty, not only will we 
  // prevent her from going even one step further (`return`), 
  // we'll go ahead and redirect her to PETA (`res.redirect`).
  if ( req.session.user.hasHistoryOfAnimalCruelty ) {
    return res.redirect('http://PETA.org');
  }

  // If the user has been seen frowning at puppies, we have to assume that
  // they might end up being mean to them, so we'll 
  if ( req.session.user.frownsAtPuppies ) {
    return res.redirect('http://www.dailypuppy.com/');
  }

  // Finally, if the user has a clean record, we'll call the `next()` function
  // to let them through to the next policy or our controller
  next();
};
```











#### view


EJS的3种模板标记：

* <%= someValue %>
* <%- someRawHTML %> 这种不会转义，比如 ' 等 
* <% /* some javascript code with access to the data as local variables */ %>



* 数据从哪里来？

在sails以及其他的一些MVC的框架里面，views的动态数据都是从controll里面传递过去的。在sails的controllers，用 res.view()这个方法可以用来传递数据。

controllers/FooController.js的index的action里面对应的view：views/foo/index.ejs。


res.view(data)，这个data参数必须是纯js对象！

比如：

```
res.view({
  name: 'Mike',
  favoriteColor: 'green',
  friends: [{
    name: 'Margaret Thatcher',
    favoriteColor: 'purple'
  },
  {
    name: 'Big Bird',
    favoriteColor: 'yellow'
  }]
});
```







#### log
