# git commit 规范
## commit message 格式
- <type>(<scope>): <subject>  
  <空行>  
  <body>  
  <空行>  
  <footer>  
> 任何一行都不得超过72个字符（或100个字符）

## 标题

### Type
- feat：新功能（feature）
- fix：修补bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- test：增加测试
- chore：构建过程或辅助工具的变动

### Scope
用来说明本次Commit影响的范围，即简要说明修改会涉及的部分。

### Subject
简要概述本次改动
- 动词开头，使用第一人称现在时
- 首字母不用大写
- 结尾不用句号(.)

## Body
<body>里面的内容是对上面subject的内容展开，在此做更加详尽的描述，内容里应该包含++修改动机++和++修改前后的对比++
- 使用第一人称现在时
- 应该说明代码变动的动机，以及与以前行为的对比

## Footer
footer里主要放置不兼容变更和Issue关闭的信息
- 不兼容变动
    如果当前代码与上一版本不兼容，则Footer要以BREAKING CHANGE开头，后面是对变动的描述、变动理由和迁移方法