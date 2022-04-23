---
title: git规范
date: 2022-04-22 18:06:37
update: 2022-04-22 18:06:37
tags: 实践
categories: 前端工程
---
# 一、提交的代码规范

`husky` + `lint-staged` + `pre-commit`

**husky:** 操作git钩子的工具

**lint-staged:** 本地暂存代码检查工具

**pre-commit:** 通过钩子函数，判断提交的代码是否符合规范
<!--more-->


1. 安装代码校验依赖

```javascript
npm i lint-staged husky -D
```

2. 在package.json中添加脚本

```javascript
"scripts": {
		"prepare": "husky install"
}
```

3. 执行 `npm run prepare` , 在根目录创建一个`husky` 文件夹

   1. ![image-20220421194607581](image-20220421194607581.png)

4. 执行 `npx husky add .husky/pre-commit "npx lint-staged"` ，在.husky文件夹下创建pre-commit shell文件 执行 npx lint-staged

   1. ![image-20220421194649682](image-20220421194649682.png)

5. 根目录创建 **.lintstagedrc.json** 文件控制检查和操作方式

   ```javascript
   {
       "*.{js,jsx,ts,tsx}": ["prettier --write .", "eslint  --fix"],
       "*.md": ["prettier --write"],
       "*.{less,css}": ["stylelint --fix"]
   }
   
   ```

# 二、提交信息规范

1. 安装提交信息依赖 [@commitlint/config-conventional](https://www.npmjs.com/package/@commitlint/config-conventional)

   ```javascript
   npm i commitlint @commitlint/config-conventional -D
   ```

2. 添加对应的shell调用方式

   ```javascript
   npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
   ```

   ![image-20220422165135542](image-20220422165135542.png)

3. 添加commitlint配置

   ```javascript
   echo "module.exports = {extends: ['@commitlint/config-conventional']};" > commitlint.config.js
   ```

   .commitlint.config.js中配置

   ```javascript
   module.exports = { extends: ['@commitlint/config-conventional'] };
   ```

4. package.json中添加

   ```javascript
       "commitlint": {
           "extends": [
               "@commitlint/config-conventional"
           ]
       },
   ```

5. 至此，就可以规范commit提交信息了。

   

   接下来，我们来了解一下commit-msg所使用到的依赖规范具体是什么样的？



## 2.1 @commitlint/config-conventional

1. **@commitlint/config-conventional** 这是一个规范配置,标识采用什么规范来执行消息校验, 这个默认是***Angular***的提交规范

2. [具体规范地址](https://www.conventionalcommits.org/en/v1.0.0/)

3. Commit message的格式

   ```javascript
   <type>(<scope>): <subject>
   <BLANK LINE>
   <body>
   <BLANK LINE>
   <footer>
   ```

   1. commit messgae分为三个部分

      1. 标题行(subject)： 必填, 描述主要修改类型和内容。
      2. 主题内容(body)：描述为什么修改, 做了什么样的修改, 以及开发的思路等等。
      3. 页脚注释(footer)：可以写注释，放 BUG 号的链接。

   2. 各配置

      1. type类型
         - feat: 新功能、新特性
         - fix: 修改 bug
         - perf: 更改代码，以提高性能（在不影响代码内部行为的前提下，对程序性能进行优化）
         - refactor: 代码重构（重构，在不影响代码内部行为、功能下的代码修改）
         - docs: 文档修改
         - style: 代码格式修改, 注意不是 css 修改（例如分号修改）
         - test: 测试用例新增、修改
         - build: 影响项目构建或依赖项修改
         - revert: 恢复上一次提交
         - ci: 持续集成相关文件修改
         - chore: 其他修改（不在上述类型中的修改）
         - release: 发布新版本
      2. scope 影响的功能或文件范围, 比如: route, component, utils, build...
      3. commit message 影响的功能或文件范围, 比如: route, component, utils, build...
      4. body 具体修改内容, 可以分为多行.
      5. Footer 一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接.

   3. 例子 

      ```javascript
      fix: prevent racing of requests
      
      Introduce a request id and a reference to latest request. Dismiss
      incoming responses other than from latest request.
      
      Remove timeouts which were used to mitigate the racing issue but are
      obsolete now.
      
      Reviewed-by: Z
      Refs: #123
      ```



# 参考

[Eslint + Prettier + Husky + Commitlint+ Lint-staged 规范前端工程代码规范](https://juejin.cn/post/7038143752036155428#heading-5)



