title: 5个有关react-redux的心得
date: 2017-06-11 09:07:28
tags: React, Redux
---

## Action Type 尽量保持与业务逻辑无关，Action Creator 可以与业务逻辑有关

action 应该尽可能细粒度，做小的、与业务无关的事情；在action creator中对action进行组合，复用。

举个例子，假设你需要一个更新用户名的 API，可以写成这样：

```javascript
// action.js
function updateUserName(userId, name) {
  return (dispatch) => {
    request.put('/api/user-name/' + userId)
    .then((user) => dispatch({
      type: UPDATE_ONE_USER,
      payload: { user },
    }));
  };
}

/////////////

// reducer.js
function user(state, action) {
  switch (action.type) {
    case UPDATE_ONE_USER: return { ...state, user: action.payload.user };
  }
}
```

这样，假如未来需要新增其他更新用户信息的API（比如变更权限），那么所要做的只是新增一个action creator，复用以前的action type即可。例如：

```javascript
// action.js
function updateUserPermission(userId, permission) {
  return (dispatch) => {
    request.put('/api/user-permission/' + userId)
    .then((user) => dispatch({
      type: UPDATE_ONE_USER,
      payload: { user },
    }));
  };
}
```

## 非共享的、局部的、UI only的状态，不一定非要放在redux state中

最典型的例子是loading状态。每个API都有独立的loading状态，甚至同一个API的每次调用都是独立的loading状态。在redux中记录这些loading信息是一件挺吃力不讨好的事情。因为首先正确记录loading状态就不是个容易的事情，很可能必须得用`API名字 + 参数`为key来记录，在使用的时候也必须小心地使用`API名字 + 参数`去拿到这个值。其次，这些loading状态可能只是一个component需要的局部UI状态，当API请求结束，loading状态的使命也宣告结束，没有存在的必要了，也就不应该存在redux里。

遇到这种问题，建议把loading放在Component内部的state中，用callback触发setState，例如：

```javascript
// action.js
function updateUserName(userId, name, callback = () => {}) { // 注意多了一个callback参数
  return (dispatch) => {
    request.put('/api/user-name/' + userId)
    .then((user) => {
      callback();
      dispatch({
        type: UPDATE_ONE_USER,
        payload: { user },
      });
    })
    .catch((err) => callback(err));
  };
}

///////////

// 具体在用的时候结合this.setState
class MyComponent extends React.Component {
  updateUserName(userId, name) {
    this.setState({ isUpdating: true }); // <-- loading开始
    this.props.updateUserName(userId, name, (err) => { // 通过callback调用setState
      this.setState({ isUpdating: false }); // <-- loading结束
    });
  }
  ...
}
```

## entities足以，idList慎用

entities指的是一种map数据结构，内容为 id -> data。例如：

```javascript
userEntities = {
  [user1.id]: user1,
  [user2.id]: user2,
  ...
};

jobEntities = {
  [job1.id]: job1,
  [job2.id]: job2,
  ...
}
```

对于列表数据，一种常见的做法是用entities存储数据，然后用idList来表示顺序。比如：

```javascript
userEntities = {
  1: user1,
  2: user2,
  3: user3,
}

ascIdList = [1, 2, 3]; // 顺序排列
descIdList = [3, 2, 1]; // 逆序排列
myIdList = [2, 3, 1]; // 其他特殊的排列顺序

// render的时候只要按照idList进行map即可：
ascUsers = ascIdList.map((id) => userEntities[id]);
descUsers = descIdList.map((id) => userEntities[id]);
myUsers = myIdList.map((id) => userEntities[id]);
```

这种组织state的方式有一个最大的问题，那就是当涉及到数据变化的时候，所有相关的idList也需要改动，不然就会发生问题。比如删掉了id是2的user，那么三个idList都要做一遍filter，去掉2。

为了避免这种数据改动导致idList被牵连改动的问题，另一种做法是完全不用idList，而是在使用的地方就地排序。比如：

```javascript
userEntities = {
  1: user1，
  2: user2,
  3: user3,
};

// 没有idList，用到的时候现场排序
ascUser = Object.keys(userEntities).sort(ascCompare).map((id) => userEntities[id]);
descUser = Object.keys(userEntities.sort(descCompare).map((id) => userEntities[id]);
myUser = Object.keys(userEntities.sort(myCompare).map((id) => userEntities[id]);
```

不过呢，上面这个做法虽然避免了idList的牵连改动，但每次使用的时候都要sort，性能上有一定影响。没关系，这个可以用memoized function解决。比如使用reselect。

所以，大部分时候用entities就够了，引入idList可能会更麻烦

## 只要你愿意，什么组件都可以connect到redux store

redux 作者推荐的一种使用redux的方式是将connect用作container组件，将具体的render逻辑放在presentation组件中。这样，data source和data presentation就可以解耦，便于替换和复用。

理论上，只要你愿意，任何组件只要connect到redux store，就可以做container组件。然而很多人认为选择哪些组件做container是十分有讲究的。有些人认为只有顶层组件才是container组件，例如APP组件；有些人认为router组件算是container组件。哪一种是对的呢？

回到问题的根本，为什么我们要求不是所有组件都可以connect呢？主要原因还是便于测试，进而提高代码可靠性。当然也有一些性能上的考虑，但在大多数情况下，性能都不是主要考虑的问题。

所以，限制只有某些组件才能做container的做法，初衷是为了保证代码单元测试起来更加容易。这其实不是一个很强力的观点，因为99%前端工程都没有单元测试，强行用这种观点去实践，反而会给开发带来额外的麻烦。

也许你见过这样的代码：

```javascript
// 这是 container 组件
class ContainerComponent extends React.Component {
  render() {
    return (
      <MiddleComponent
        a={this.props.a}
        b={this.props.b}
        c={this.props.c}
        d={this.props.d}
      />
    );
  }
}

class MiddleComponent extends React.Component {
  render() {
    return (
      <InnerComponent
        a={this.props.a} // <-- a、b、c、d这些属性对MiddleComponent完全没有意义，只是原封不动地传给了InnerComponent
        b={this.props.b}
        c={this.props.c}
        d={this.props.d}
      />
    );
  }
}
```

看到这种代码，我立刻就想到了“酒肉穿肠过，佛祖心中留”。借用王银曾经评论haskell的比喻，这就好比是一个没有无线电的世界，所有的信号传递都只能通过一根根电线链接两头。

为了避免上面那种壮丽的景象，有人选择这么写：

```javascript
class MiddleComponent extends React.Component {
  render() {
    return (
      <InnerComponent {...this.props}/>
    );
  }
}
```

这种写法看似简单了很多，但由于props的传递是隐式的，开发者要弄清究竟传了什么，就必须回溯到最开始的container组件。这就好比一些数学教材中的证明答案：由第xx题的证明结论，可以得到blablabla。但是你去看第XX题，发现也是一样的：由第YY题的证明结论可以得到blablabla。然后你又要去看YY题的证明过程，可能得到的还是类似的东西。。

再多说一句，凡是隐式的、动态的写法，基本都是有害的。因为写的时候你爽了，以后其他人维护代码的时候就会被坑惨了。

所以，不要迷信什么必须只有顶层容器才能connect之类的best practice，大胆connect吧，只要别再common组件里connect就行（common组件不要混入业务逻辑，也就不要connect）。

## 避免嵌套过深的state结构

这个道理很容易理解，越是嵌套深，越是不容易修改（因为要return新的state），不懂的话吃过亏也就懂了。
