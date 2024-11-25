---
title: JavaScript实现本地文件扫描
category: Programming
tags: [JavaScript, 前端, 算法]
abbrlink: JS-LocalFileScan
published: 2024-11-22 00:00:00
ai:
  - 本文介绍了使用JavaScript实现本地文件扫描的方法。
---

跟一个老前端在做一个`NAS`云存储的工具，涉及到了本地文件扫描，作为一个前端菜鸡，我去了解了一下功能的具体原理，然后自己使用`JavaScript`实现了一下，现在做一个记录。

## 原理

说白了，就是普通的递归查找，输入所需要查找的文件目录，使用`Node`的`fs`模块读取目录，然后递归查找子目录，通过`isDirectory`方法判断子目录中是否也存在文件夹，然后一直重复即可，直到找到所有文件。

## 预期结果

我是想返回一个类似于`JSON`的变量，预期的结构为:

```json
{
    "根目录": {
        {"子目录1":"floder"},
        {"子目录2":"floder"},
        {"文件1":"file"}, 
        {"文件2":"file"}
        },
    "子目录1": {
        {"文件1":"file"}, 
        {"文件2":"file"},
        {"子目录3":"floder"}
        },
    "子目录2": {
        {"文件1":"file"}, 
        {"文件2":"file"}
        }
}
```

在这个类似于`JSON`变量中，每一个`Key`代表着一个目录，对应的`Value`对应着这个目录下的所有文件和子目录，并且都有标签与其对应，`floder`表示这是一个目录，而`file`表示这是一个文件。这样的标记有利于后续在前端页面中展示的时候对图标的选择等。

## 代码实现

写来写去，为了让最后结果能够**集大成**，选择了使用`柯里化 Currying`和`闭包`来实现这个功能。

首先定义一个父函数`scanFile`，它声明一个`fileList`变量，用来保存最后的结果。

```javascript
async function scanFile() {
  let fileList = {};
}
```

然后定义一个子函数`scan`，用来递归查找目录。

```javascript
async function scan(dir){
    let files
    try{
      files = await readdir(dir, { withFileTypes: true });
    }catch{
      return
    }
    fileList[dir] = {};
    for (const file of files){
      if (file.isDirectory()) {
        const sonFilePath = `${dir}${file.name}/`;
        fileList[dir][file.name] = "floder"
        await scan(sonFilePath);
      }else{
        fileList[dir][file.name] = "file"
      }
    }
    return fileList
  }
```

上述代码中，`readdir`是`Node`的`fs`模块中的方法，用来读取目录，`withFileTypes: true`表示返回的是一个`Dirent`对象，这个对象中包含了文件名、文件类型等信息。使用`files`保存读取到的文件列表，然后遍历这个列表，如果当前遍历到的文件是一个目录，则递归调用`scan`函数，否则将文件名和类型添加到`fileList`中。最后返回`fileList`。

然后，在`scanFile`函数中返回`scan`函数。
完整代码:

```javascript
import { readdir } from 'node:fs/promises';

async function scanFile() {
  let fileList = {};
  async function scan(dir){
    let files
    try{
      files = await readdir(dir, { withFileTypes: true });
    }catch{
      return
    }
    fileList[dir] = {};
    for (const file of files){
      if (file.isDirectory()) {
        const sonFilePath = `${dir}${file.name}/`;
        fileList[dir][file.name] = "floder"
        await scan(sonFilePath);
      }else{
        fileList[dir][file.name] = "file"
      }
    }
    return fileList
  }
  return scan
}
```

可以尝试打印一下结果。

```javascript
const result = await (await scanFile())("D:/git/");
console.log(result);
```

输出为:

```JSON
{
  'D:/git/': { 
    'Git-2.45.2-64-bit.exe': 'file', 
    xxx: 'folder' 
    },
  'D:/git/xxx/': {
    hhh: 'folder',
    'hhh.zip': 'file',
    'xh.bmp': 'file',
    'xxx.pptx': 'file',
    'xxx.txt': 'file'
  },
  'D:/git/xxx/hhh/': {}
}
```

可以看到，`scanFile`函数返回了一个函数，这个函数返回了一个`fileList`对象，这个对象中保存了所有目录和文件的信息。

本地目录结果为:
![本地目录](https://images.kimbleex.top/BlogIMG/JS_LocalFileScan/local.avif)
