title: 'Meteor七牛客户端上传的例子'
date: 2016-02-01 16:44:53
tags:
---

## 准备工作

1. 安装七牛 nodejs SDK

  因为 Meteor 无法使用原生 npm 包，必须借助 [meteorhacks](https://github.com/meteorhacks/npm) 才能使用，所以确保已经安装了 meteorhacks 。

  首先安装 meteorhacks

  `meteor add meteorhacks:npm`

  然后在项目根路径下编辑 packages.json 文件，增加七牛的 node SDK

  ```
  {
    "qiniu": "6.1.9"
  }
  ```

  完成后重启 meteor 服务器即可

  `meteor`

  > 偶尔 meteorhacks 会出现无法正确加载 node modules 的问题，如果是这样尝试先卸载 meteorhacks 然后重新安装。

2. 安装七牛 js SDK

  > 前面安装的 nodejs SDK 是后端依赖，现在安装的 js SDK 是前端依赖，二者不要搞混了。

  根据七牛文档说明，需要下载两个东西：plupload 和 七牛的 js SDK，下载后放在相应的路径下（比如 public ）然后让 index.html 引入即可。

3. 安装 Meteor iron router

  因为七牛上传需要服务器端和前端配合，需要配置好路由，所以要安装 [iron router](https://github.com/iron-meteor/iron-router)

  `meteor add iron:router`

## 七牛上传流程

首先再回顾一下七牛官方教程中提到的文件上传流程

{% asset_img qiniu-upload-flow.png 七牛上传流程 %}

> 业务服务器指的是 Meteor 所在服务器

其中，步骤 1 和 2 需要借助前面安装的七牛 nodejs SDK 实现，而步骤 3 和 4 则需要借助 js SDK实现。

## 服务器端

首先从服务器端开始，其实要做的就一件事：响应 token 请求

利用 iron router，配置好相应路由 method

```javascript
const qiniu = Meteor.npmRequire('qiniu');
qiniu.conf.ACCESS_KEY = '你的 ACCESS_KEY';
qiniu.conf.SECRET_KEY = '你的 SECRET_KEY';

Router.route('/api/uptoken', { where: 'server' })
.get(function() {
  const res = this.response;
  const token = new qiniu.rs.PutPolicy('你的bucket名字');

  res.statusCode = 200;
  res.end(JSON.stringify({ uptoken: token }));
});
```

这样，前端就可以通过向 '/api/uptoken' 这个 url 请求 upload token 了

当然，这里只是出于演示的目的，简化了 token 的生成过程。具体的细节请参考七牛的 [nodejs SDK 文档](http://developer.qiniu.com/docs/v6/sdk/nodejs-sdk.html)

## 客户端

客户端有两件事要做

1. 向后端请求 upload token

  只需要向之前定义的 url 发送 GET 请求即可。方法有很多种，这里就不罗嗦了。

2. 拿着 upload token 向七牛服务器上传文件

  这里要用到七牛的 js SDK。因为只是演示，所以就直接复制 js SDK 文档上的代码了。

  别忘了在 html 页面中放置一个 id 为 pickfiles 的 button ，而且其父容器 id 为 container。这是 plupload 所需要的，你也可以在 SDK 的配置中自定义 id 。

```javascript
Qiniu.uploader({
  runtimes: 'html5,html4',          //上传模式,依次退化
  browse_button: 'pickfiles',       //上传选择的点选按钮，**必需**
  uptoken_url: '/api/uptoken',      //Ajax请求upToken的Url，**强烈建议设置**（服务端提供）
  domain: 'http://你的七牛域名',      //bucket 域名，下载资源时用到，**必需**
  get_new_uptoken: false,           //设置上传文件的时候是否每次都重新获取新的token
  container: 'container',           //上传区域DOM ID，默认是browser_button的父元素，
  max_file_size: '100mb',           //最大文件体积限制
  max_retries: 3,                   //上传失败最大重试次数
  dragdrop: true,                   //开启可拖曳上传
  drop_element: 'container',        //拖曳上传区域元素的ID，拖曳文件或文件夹后可触发上传
  chunk_size: '4mb',                //分块上传时，每片的体积
  auto_start: true,                 //选择文件后自动上传，若关闭需要自己绑定事件触发上传,
  init: {
    FilesAdded: function(up, files) {
      plupload.each(files, function(file) {
        // 文件添加进队列后,处理相关的事情
      });
    },
    BeforeUpload: function(up, file) {
      // 每个文件上传前,处理相关的事情
    },
    UploadProgress: function(up, file) {
      // 每个文件上传时,处理相关的事情
    },
    FileUploaded: function(up, file, info) {
      // 每个文件上传成功后,处理相关的事情
      // 其中 info 是文件上传成功后，服务端返回的json，形式如
      // {
      //    "hash": "Fh8xVqod2MQ1mocfI4S4KpRL6D98",
      //    "key": "gogopher.jpg"
      //  }
      // 参考http://developer.qiniu.com/docs/v6/api/overview/up/response/simple-response.html

      // var domain = up.getOption('domain');
      // var res = parseJSON(info);
      // var sourceLink = domain + res.key; 获取上传成功后的文件的Url
    },
    Error: function(up, err, errTip) {
      // 上传出错时,处理相关的事情
    },
    UploadComplete: function() {
      //队列文件处理完毕后,处理相关的事情
    },
    Key: function(up, file) {
      // 若想在前端对每个文件的key进行个性化处理，可以配置该函数
      // 该配置必须要在 unique_names: false , save_key: false 时才生效

      var key = "";
      // do something with key here
      return key
    },
  },
});
```

## 大功告成！

作完上面的工作后，你就可以上传文件了，赶紧试试吧
