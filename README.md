[git仓库]: https://github.com/GuoBinyong/library-rollup-template
[issues]: https://github.com/GuoBinyong/library-rollup-template/issues

[码云仓库]: https://gitee.com/guobinyong/library-rollup-template


[library-webpack-template]: https://github.com/GuoBinyong/library-webpack-template

[Git并行工作流程规范]: https://www.jianshu.com/p/d7a3a4935440


> 构建过程是复杂的前端项目必不可少的环节；但 业务代码 与 公用代码（如：封装的库、组件、工具等被复用的代码）的构建需求是不一样的；我发现很多 npm 贡献者都没意识到这一点，他们用传统的业务项目的 webpack 配置 去打包 公用代码，虽然能运行，但实际潜藏着许多问题，甚至都不经过编译构建，直接发布单纯的源码到 npm 上；由于本人经常封装一些东西，为了方便，就分别使用 rollup 和 webpack 开发并配置了专门用于构建公共代码的配置模板；本仓库是 rollup 版本的配置模板；webpack 版本的配置模板请看 [library-webpack-template][]


目录
=======


<!-- TOC -->

- [1. 简介](#1-简介)
- [2. 分支介绍](#2-分支介绍)
- [3. 使用](#3-使用)
- [4. 命令](#4-命令)
- [5. 所有可能需要的更改](#5-所有可能需要的更改)
    - [5.1. 更改库的相关信息](#51-更改库的相关信息)
    - [5.2. 禁止输出某种模块的包](#52-禁止输出某种模块的包)
- [6. 业务代码与公用代码的构建特点](#6-业务代码与公用代码的构建特点)
    - [6.1. 业务代码](#61-业务代码)
    - [6.2. 公用代码](#62-公用代码)
- [7. 构建工具的选择](#7-构建工具的选择)
- [8. 公共代码构建的配置目标](#8-公共代码构建的配置目标)
- [9. 组织结构](#9-组织结构)
- [10. npm包管理配置文件](#10-npm包管理配置文件)
- [11. Rollup配置文件](#11-rollup配置文件)
- [12. TypeScript配置文件](#12-typescript配置文件)
- [13. 代码检查](#13-代码检查)
- [14. 项目文档Readme](#14-项目文档readme)

<!-- /TOC -->


内容
=======


**注意：**
为了方便下文描述，我把 **业务代码** 构建出的最终产品称为 **应用程序**；把 **公用代码** 构建的出产品称为 **组件**；



# 1. 简介
library-rollup-template 称为 库构建模板，又称 公共代码构建模板 ，是专门用于构建 公共代码（如：封装的库、工具等） 的 rollup 配置模板；针对不同的开发环境（如：TypeScript、JavaScript 等等）library-rollup-template 仓库以分支的方式提供了多种配置模板，当需要开发和构建公共代码时，直接下载相应分支，并默认在的 src 目录下开发即可。



**具有以下特性：**  
- 易使用：一般情况下，只需简单简单更改下 `package.json` 文件的相关信息即可；
- 可配置去除 公共代码 的依赖，除了 `iife` 格式的包，默认情况下都去除了 所有 `node_modules` 中的依赖；
- 多个构建目标可同时构建，即：一次构建分别生成多个构建目标的构建包，如：`ESModule`、`CommonJS`、`AMD`、`UMD`、`自执行文件`（可用 `<script>` 标签直接引入） 等等；
- 在构建的输出包的顶部会自动注入 包 和 作者的相关描述信息；
- 可快速切换 开发模式 和 生产模式；
- 支持 JavaScript、JSX、TypeScript、TSX 等等；
- 支持 ESLint；
- 配置文件中有详细丰富的注释说明，学习成本低；



**详情请看：**  
- 主页：<https://github.com/GuoBinyong/library-rollup-template>
- [GitHub仓库][git仓库]
- [码云仓库][]


**如果您在使用的过程中遇到了问题，或者有好的建议和想法，您都可以通过以下方式联系我，期待与您的交流：**
- 给该仓库提交 [issues][]
- 给我 Pull requests
- 邮箱：<guobinyong@qq.com>
- QQ：guobinyong@qq.com
- 微信：keyanzhe


# 2. 分支介绍
针对不同的开发环境（如：TypeScript、JavaScript 等等），library-rollup-template 有多种配置模板，为了后续维护升级方便 和 体现种模板间的差异，本仓库采用如下策略来管理各个模板：
- 为同类型配置模板创建相同的分支路径，如：`js/`、`ts/`
- 为每类配置模板创建公共分支 `common`，如：`common`、`js/common`、`ts/common`
- 每类具体的配置模板配置都是基于  其基础分支来创建 或 该类的公共分支 `common`
- 当需要更改某一类下的所有配置模板时，需要在该类下的 公共分支 `common` 中更改，然后将更改合并到其直接子分支，然后再将直接子分支合并到子分支的直接子分支中，依次类推；（为了保持分支的线性、美观，我使用的是变基rebase的合并方式，如果您们也使用该分支模型，则不建义使用变基的方式进行合并，推荐使用 merge）

_提示：关于`子分支`、`母分支`的概念，请参看[Git并行工作流程规范][]_


所以，本仓库的分支结构如下：
```
common  # 仓库的公共分支，即：所有分支的公共分支；不能用作公共代码的构建模板；
js/      # 针对 JavaScript 开发环境的配置模板 都放在这个分支路径下
  ├── common    # 所有 JavaScript 开发环境配置模板的公共配置；不能用作公共代码的构建模板；基于 common 分支创建；
  ├── babel/    # 用 babel 作为 编译器的配置模板 都放在这个分支路径下
  │     ├── base    # 仅支持已规范的特性，不支持处于实验阶段的特性；基于 js/common 分支创建；
  │     └── stage    # 基于 base 创建，支持已规范的特性 和 处于实验阶段的特性；基于 js/babel/common 分支创建；
  └── buble    # 用 buble 作为 编译器的配置模板
ts/      # 针对 TypeScript 开发环境的配置模板 都放在这个分支路径下
  ├── common    # 所有 TypeScript 开发环境配置模板的 公共配置；不能用作公共代码的构建模板；基于 common 分支创建；
  ├── tsc    # 用 TypeScript编译器 作为 编译器的配置模板；基于 ts/common 分支创建；
  └── ts-js/   # 针对 TypeScript 和 JavaScript 混合开发环境的配置模板 都放在这个分支路径下；即：项目中即有 TypeScript 代码，又有 JavaScript 代码；即可用于纯 TypeScript 项目，也可用于 混合开发项目；
        ├── common    # 所有混合开发环境配置模板的 公共配置；不能用作公共代码的构建模板；基于 ts/common 分支创建；
        ├── babel    # 用 babel 作为 编译器的配置模板；基于 ts/ts-js/common 分支创建；
        └── tsc-babel   # 用 TypeScript编译器 编译后 再经过 babel编译器 进行编译 的配置模板；基于 ts/ts-js/babel 分支创建；
```

**说明：**  
`ts/ts-js/` 分支路径下都是 针对 TypeScript 和 JavaScript 混合开发环境的配置模板；混合开发环境一般常见于 JavaScript 旧项目，但新的模块需要使用 TypeScript 进行开发的场景；

若想能让 TypeScript 文件能够更加准确感知 JavaScript 文件的类型，你可以为 JavaScript 文件书写类型声明文件(即：后缀为 `.d.ts` 的文件)；默认配置是建义你将 JavaScript 的 类型声明文件`.d.ts` 都放在 `types/` 目录下；

# 3. 使用
1. 从 [git仓库][] 下载相应的分支到本地；
2. 将 `package.json` 文件中 `library-rollup-template` 库和作者相关的信息替换为您期望的信息；_注意：`files` 以上的配置项都是你需要根据自己的库进行定制的配置_
3. 然后默认从 `src/index` 文件开始写代码；




# 4. 命令
library-rollup-template 项目支持如下构建命令：
- `npm run build` : 构建项目；
- `npm run dev` : 构建项目，并监听源文件的改动，如果有改动，便会重新打包；
- `npm run test` : 执行 `test/index` 文件；


如果已经全局安装了 `rollup` 命令，也可以在项目根目录下执行如下命令：
- `rollup -c` : 构建项目；
- `rollup -c -w` : 构建项目，并监听源文件的改动，如果有改动，便会重新打包；




# 5. 所有可能需要的更改


## 5.1. 更改库的相关信息
- 库的相关信息基本上都包含在 `package.json` 文件中，其中 `files` 以上的配置项应该都是你需要根据自己的库进行定制的配置；
- 作者的相关信息存在 `package.json` 和 `LICENSE` 这两个文件中；

**技巧：**  
1. 全局将 `library-rollup-template` 字符串 替换成 你的包名；
2. 全局将 `郭斌勇` 字符串替换成 您的名字；
3. 全局将 `guobinyong@qq.com` 字符串 替换成 您的邮箱；



## 5.2. 禁止输出某种模块的包

library-rollup-template 默认构建输出了以下几种模块格式的包：
- `es`：ESModule
- `cjs`：CommonJS
- `amd`：AMD
- `umd` : umd
- `iife`：自执行

输出的包名的格式是 `<包名>.<格式>.js`，如果不需要输出某种格式的包，只需将对应格式的配置块注释掉即可；




# 6. 业务代码与公用代码的构建特点

## 6.1. 业务代码
- 需要把所有代码及依赖（包括 公用代码）都构建在一块，作为一个整体来运行；
    因为：业务代码的最终呈现效果是应用程序，应用程序是一个完整的代码逻辑，任何依赖的缺失都会导致应用程序崩溃。

- 运行环境单一；
    业务代码构建的应用程序要么是运行在浏览器的，要么是运行在 node 环境的；因为业务产品是最终供大众用户使用的，在开发业务产品（应用程序）之初，就已确定了业务产品的运行环境。

- 构建目标单一；
    在构建业务代码时，往往构建目标是明确的，要么是构建浏览器应用，要么是构建 note 应用，或者其它，总之很少有同时构建多个目标的，比如同时构建浏览器和 note 环境的应用程序。

- 有较强的包体积的限制；
    因为大部分应用程序是运行在浏览器的，较大的包体积会使应用程序加载时间过长，从而影响用户体验，所以，一般应用程序都要求包的体积尽可能的小；

- 业务代码中通常包含 HTML、CSS、JavaScript 文件；
- CSS 和 JavaScript 文件通常都需要在 HTML 文件中引入；
    浏览器端的应用程序都是以HTML文件为入口的，通过HTML加载 CSS 和 JavaScript 文件；


## 6.2. 公用代码
- 需要去除依赖；
    公共代码的依赖往往也是其它公共代码或者业务代码的依赖，当业务代码中引入公共代码时，极有可能也引入了该公共代码的依赖，如果公共代码中不去除其依赖，则会导致业务代码中包含多份公共代码的依赖，造成代码冗余，增大业务代码的体积；

- 运行环境多样；
    公众代码是被应用程序引用的，应用程序的运行环境可能是浏览器，也可能是 node ，或者其它 ，所以公共代码的运行环境是多样性的；

- 构建目标多样；
    因为公共代码的运行环境是多样的，所以在需要对公共代码进行构建时，往往需要针对每个运行环境分别进行构建；

- 对包体积的没有特别强列的要求；
    公共代码最终是要被业务代码引用的，面业务代码构建成应用程序时通常需要做包体积的压缩的，所以，对包体积的压缩通常会在业务代码构建成应用程序时进行，所以，当公共代码构建成组件时，大多数情况下也可不做体积压缩；

- 通常不包含 HTML 文件；
- CSS 和 JavaScript 文件通常分开；
    CSS 和 JavaScript 的组织方式往往是由业务代码组织结构和构建方案决定，所以，在公共代码中，CSS 和 JavaScript 通常是分开的，具体怎么组织，由业务代码决定；



# 7. 构建工具的选择
前端的构建工具有很多，像：Webpack、rollup、Browserify、Parcel、Grunt、Gulp等等；

目前，对于构建公共代码的工具较常用的是 rollup，对于构建业务代码，较常用的工具是 Webpack；不过，Webpack 也是可以用于构建公共代码的，详见[library-webpack-template][]。


相比 rollup ，要构建库方面，Webpack有以下缺点：
- 配置较复杂
- 不支持构建 ESModule 模块的包（解释：Webpack 可以编译 ESModule 模块的源文件，但不支持输出 ESModule 格式的包）；

所以，推荐使用 rollup 构建 库；本项目就是使用 rollup 来搭建的模板；

如果想用 Webpack 来构建库，可以使用另一个模板 [library-webpack-template][]；


# 8. 公共代码构建的配置目标
公共代码构建的配置目标其就是实现上文所述的公共代码的构建特点，总结如下：

- 去除依赖
- 一次构建可分别生成适用于不同环境的码包；
    比如：分别生成用于 node 和 浏览器 环境的包，或者 不同模块化方案的包，如 AMD、CMD、 CommonJS、UMD等等；
- 能在 开发 和 生产 两种模式快速切换；
    开发 模式下地在需要更多的调试信息，如 Source Map ；而生产模式下需求尽可能地压缩包的体积；在开发调试的过程中，需要在开发模式下构建包；当开发完毕，准备发布时，需要在生产模式下构建包；
- 分离 CSS 和 JavaScript 文件；





# 9. 组织结构
模板项目中默认包含了一些文件和目录，它们的组织结构和作用如下所示：
```
library-rollup-template/   # 构建前端库的webpack打包配置模板项目根目录
   ├── package.json            # npm 的包管理配置文件
   ├── src/                    # 默认的包含项目源代码的目录
   │   └── index               # 默认的构建入口文件
   ├── test/                   # 默认的包含测试代码的目录
   │   └── index               # 默认的构建入口文件
   ├── dist/                   # 默认的构建输出目录
   ├── LICENSE                 # 开源证可证；默认是 MIT 许可证
   ├── tsconfig.json           # TypeScript 的配置文件
   ├── README.md               # 项目的说明文档
   ├── .eslintignore           # ESLint 的忽略配置文件
   ├── .eslintrc.js            # ESLint 的配置文件
   ├── .gitignore              # git 的忽略配置文件
   └── .npmignore              # npm 上传包时的忽略配置文件
```



# 10. npm包管理配置文件
library-rollup-template 中与 npm 包管理相关的配置文件有 2 个：
- `.npmignore` : npm 上传包时的忽略配置文件；默认忽略了 与构建配置文件的所有文件和目录，如：`build/` 等等；也忽略了开发环境默认的输出目录 `dev/`，还有编辑器相关的文件和目录，如：`.idea`、`.vscode`；
- `package.json` : npm 的包管理配置文件；同时也是 通过 npm 上传、发布 包 的 配置模板文件；

**注意：** `package.json` 文件中的如下字段：
- `module` : 该字段是用来指定以标准模块暴露的出口文件；
- `types` | `typings` ： 该字段是用来指定库的类型声明文件；如果库没有类型声明文件，则去除该字段；
- `"sideEffects" : boolean | Array<string>` ，可以为布尔，表示整个包是否有副作用；也可以是一些有副作用文件的的路径字符串，路径支持 相对路径、绝对路径和 glob 模式； 副作用标记；表明项目中的哪些文件不是 纯的 ES2015 模块，由此不能安全地删除文件中未使用的部分，"side effect(副作用)" 的定义是，在导入时会执行特殊行为的代码，而不是仅仅暴露一个 export 或多个 export。举例说明，例如 polyfill，它影响全局作用域，并且通常不提供 export；详细内容请见 <https://webpack.docschina.org/guides/tree-shaking/#将文件标记为-side-effect-free-无副作用->

_关于`package.json`文件的详细配置信息请参考<https://docs.npmjs.com/files/package.json>_



# 11. Rollup配置文件
library-rollup-template 中的 Rollup 配置文件是`rollup.config.js`，里面有详细清晰的注释，能让你轻松入手；

_关于 Rollup 的详细配置信息请参考<https://rollupjs.org>_


# 12. TypeScript配置文件
如果你使用的是 TypeScript 的模板，就会包含一个 TypeScript 配置文件 `tsconfig.json`，所以，你也可以直接在项目根目录下直接使用 TypeScript 的编译命令 `tsc`；

_关于 `tsconfig` 的详细配置信息请参考 <https://www.typescriptlang.org/docs/handbook/tsconfig-json.html>_


# 13. 代码检查
本配置模版默认使用 ESLint 作为 JavaScript 和 TypeScript 的代码检查工具；至于为什么 TypeScript 不用 TSLint 作为代码检查工具的原因请参看 <https://github.com/typescript-eslint/typescript-eslint>；

ESLint 相关的配置文件如下：
- `.eslintrc.js` : ESLint 的配置文件，可配置 解析器、解析选项、规则 等等；默认是 TypeScript 代码检查配置；
- `.eslintignore` : ESLint 的忽略文件的配置文件；

_关于 ESLint 的详细配置信息请参考 <http://eslint.cn/docs/user-guide/configuring>_




# 14. 项目文档Readme
作为让别人作用的项目，一定要有文档，优化库；`README.md` 是专门用来介绍项目的说明性文档；

我个人认为，库的文档至少需要以下几个：
- 介绍、说明性文档：用于介绍、说明项目的文档；该文档一般写在 `README.md` 文件中；
- 教程文档：以易学、易懂为原则的教学型文档；
- 接口文档：以规范、全面、方便查找为原则的描述接口的手册型文档；

其中 `README.md` 写在项目的根目录里，并建义在 `README.md` 中留有其它文档的入口；其它文档（如：教程文档、接口文档）建义放在 `doc/` 或 `docs/` 目录里；


我个人推荐的 `README.md` 目录结构如下：

+ 简介：库的简单介绍
   - 特性简介：库的特性
   - 相关链接：如主页、仓库、相关文档等等
   - 反馈问题的各种方式：如：issues、作者的联系方式 等等
+ 安装：库的安装方式
+ 简单的使用教程：库的简单使用教程









--------------------

> 有您的支持，我会在开源的道路上，越走越远

![赞赏码](https://i.loli.net/2020/04/08/PGsAEqdJCin1oQL.jpg)
