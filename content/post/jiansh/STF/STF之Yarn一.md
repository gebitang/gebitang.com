+++
title = "STF之Yarn一"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "STF"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/7201f9ba28c3)

Mac环境部署告一段落，同事多次建议为什么不用yarn。STF本身也包含了yarn.lock文件。简单学习一下[yarn](https://github.com/yarnpkg/yarn/)
的相关内容。

>Yarn is a package manager for your code. Code is shared through something called a **package** (sometimes referred to as a **module**). A package contains all the code being shared as well as a `package.json` file which describes the package.

- 不建议使用brew install yarn
brew会自动安装最新的node依赖（目前为node12版本），但其实我们只需要yarn而已。

- 使用shell脚本安装
`curl -o- -L https://yarnpkg.com/install.sh | bash` 

[dependency-types](https://yarnpkg.com/en/docs/dependency-types)
>Normal dependencies are usually installed from the npm registry.

所以yarn也是使用npm的registry。

```
yarn --version

# 根据package.json文件进行全量安装
yarn install
yarn
```

- Adding a dependency
```
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]
```

- Add to devDependencies, peerDependencies, and optionalDependencies respectively:
```
yarn add [package] --dev
yarn add [package] --peer
yarn add [package] --optional
```

- Upgrading a dependency
```
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]
```

- Remove
```
yarn remove [package]
```
