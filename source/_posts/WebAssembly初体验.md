---
title: WebAssembly初体验
date: 2020-01-07 23:02:21
tags: WebAssembly
---
### 一、编译wasm

#### 1.1 准备环境

* 安装git

* 安装python

* 准备Emscripten SDK

  ```git
    $ git clone https://github.com/emscripten-core/emsdk.git
    $ cd emsdk
    $ ./emsdk install latest
    $ ./emsdk activate latest
    
    $ source ./emsdk_env.sh --build=Release  //本命令行窗口可以直接使用emcc命令
    ```



#### 1.2 编辑wasm文件

* 编写math.c文件

  ```c
  int add(int a,int b){
      return a+b;
  }
  
  int minus(int a,int b){
      return a-b;
  }
  
  int multiply(int a,int b){
      return a*b;
  }
  ```

* 通过emsdk生成math.wasm

  ```javascript
  //注意emcc命令，需要事先source ./emsdk_env.sh --build=Release
  $ emcc math.c -Os -s WASM=1 -s SIDE_MODULE=1 -o math.wasm
  ```

* 这样我们的`math.wasm`文件就生成好了

### 二、js引入wasm

#### 2.1 初始化一个工程目录

略，注意webpack的版本必须是4及以上,

#### 2.2 引入loader和fetch

  ```javascript
  npm install wasm-loader --save-dev
  npm install node-fetch  --save-dev
  ```

#### 2.3 编写引入函数和测试函数

  ```javascript
  const getExportFunction = async (url) => {
      const env = {
        memoryBase: 0,
        tableBase: 0,
        memory: new WebAssembly.Memory({
          initial: 256
        }),
        table: new WebAssembly.Table({
          initial: 2,
          element: 'anyfunc'
        })
      };
      const instance = await fetch(url).then((response) => {
        return response.arrayBuffer();
      }).then((bytes) => {
        return WebAssembly.instantiate(bytes, {env: env})
      }).then((instance) => {
        return instance.instance.exports;
      });
      return instance;
    };

  const test = async () => {
      const wasmUrl = 'http://localhost:3000/math.wasm';
      const { add,minus,multiply } = await getExportFunction(wasmUrl);
      console.log('200+100=',add(200,100));
      console.log('200-100=',minus(200,100));
      console.log('200*100=',multiply(200,100));
      alert('打开控制台查看');
    };

  //测试执行
  test();
  ```

打开控制台就能看到下面的内容了

  ```javascript
  200+100= 300
  200-100= 100
  200*100= 20000
  ```



#### 2.4 其它

上述的方法是将wasm作为一个外部的静态资源文件加载；

若要降wasm作为内部文件加载，仅仅需要通过配置webpack即可；

思考，能不能载工程源码中直接编写c或c++代码呢，然后统一编译。



参考：

[https://webassembly.org/getting-started/developers-guide/](https://webassembly.org/getting-started/developers-guide/)

[https://segmentfault.com/a/1190000008402872](https://segmentfault.com/a/1190000008402872)

[https://www.cnblogs.com/detectiveHLH/p/9881626.html](https://www.cnblogs.com/detectiveHLH/p/9881626.html)