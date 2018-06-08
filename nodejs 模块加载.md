
```javascript
require('http') //#1
require('./script') //#2
require('./script.js') //#3
require('./script.node') //#4
require('./script.json') //#5
require('script') //#6
```

以上几种不同的 require 方式
  
    1. 先从缓存里寻找
  
    2. 判断是不是个内置的全局模块 #1
  
    3. 是不是路径开头 / ../ ./
  
    3.1 如果带 .js .node .json 后缀，按路径加载 #345
    
    3.2 如果不带 #2
    
      3.2.1 直接找到了无后缀的文件，加载
      
      3.2.2 如果没有，尝试加上 .js .json .node 加载, 变成 #345
      
      3.2.3 还是没有，猜测可能是个目录，尝试进入
      
        3.2.3.1 进这个目录 尝试获取 ./script/package.json 中的 main 字段
        
        3.2.3.2 尝试 ./script/index.json ./script/index.js ./script/index.node
        
    4. 当前目录 node_modules 中找，寻找方式类似 3.2.3 的目录；找不到就往父目录的 node_modules 找
  
    5. 算了，找不到了，抛异常
    
    
*缓存的标记都是按模块绝对路径来计算的*
