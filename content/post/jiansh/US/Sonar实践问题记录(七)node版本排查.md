+++
title = "Sonar实践问题记录(七)node版本排查"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "US"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/b63ca466b644)

[关键问题 #!/usr/bin/env node](https://stackoverflow.com/a/33510581/1087122)

采用事实上标准而不是通用标准。所以前端项目扫描使用ESlint，而不是Sonar自带的扫描规则。（如何使用的逻辑还没有搞清楚，使用的环境已是ready状态）。

前端同事已经封装了eslint接口，只需要提供扫描的项目目录和输出地址，就可以完成扫描动作。这对我的接入就非常方便了。

问题是：“版本太低，语法不支持。”
```
        ...calculateStatsPerFile(messages)
        ^^^

SyntaxError: Unexpected token ...
    at createScript (vm.js:56:10)
    at Object.runInThisContext (vm.js:97:10)
    at Module._compile (module.js:549:28)
    at Object.Module._extensions..js (module.js:586:10)
    at Module.load (module.js:494:32)
    at tryModuleLoad (module.js:453:12)
    at Function.Module._load (module.js:445:3)
    at Module.require (module.js:504:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/root/.nvm/versions/node/v10.13.0/lib/node_modules/eslint-my-server/node_modules/eslint/lib/cli-engine/index.js:3:23)
```

登录到服务器检查，`node -v`显示版本为`v10.13.0`已经足够新。前端建议在调用写的js脚本前指定node目录，类似`/root/.nvm/versions/node/v10.13.0/bin/node`

先手动验证，~~这样是无效的~~。这样是有效的，但在系统执行时不知是否依然有效。因为手动验证时，即使不手动指定node，依然可以正常执行。

类似下面这两个执行方式都可以获得到结果——
```
# with node prefix
/root/.nvm/versions/node/v10.13.0/bin/node /root/.nvm/versions/node/v10.13.0/bin/eslint-my-server -p project-path -o result.json
# without node prefix
/root/.nvm/versions/node/v10.13.0/bin/eslint-my-server -p project-path -o result.json
```

检查一下环境，已经js脚本实际的内容——

```
nod[root@testme]# node -v
v10.13.0
[root@testme]# which node
/root/.nvm/versions/node/v10.13.0/bin/node
[root@testme]# whereis node
node: /usr/bin/node /usr/share/node /root/.nvm/versions/node/v10.13.0/bin/node
[root@testme]# ll /root/.nvm/versions/node/v10.13.0/bin/eslint-cars-server 
lrwxrwxrwx. 1 root root 51 Dec 13 17:17 /root/.nvm/versions/node/v10.13.0/bin/eslint-my-server -> ../lib/node_modules/eslint-my-server/bin/index.js
[root@testme]# cat /root/.nvm/versions/node/v10.13.0/lib/node_modules/eslint-my-server/bin/index.js 
#!/usr/bin/env node
```
js脚本指定了 `#!/usr/bin/env node`，默认会检查`/usr/bin/`目录下的node？ 

root启动的java应用中的环境与 terminal登录进去的环境似乎有差异？

写一个简单的java程序检查一下几个node（保存为nodecheck.java文件）
```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

class NodeCheck {

    public static void main(String[] args) {
        check("node -v");
        check("/usr/bin/node -v");
        check("/root/.nvm/versions/node/v10.13.0/bin/node -v");
    }

    private static void check(String command)  {


        StringBuilder sb = new StringBuilder();
        String line;
        try  {
            Process exec = Runtime.getRuntime().exec(command, null, null);
            BufferedReader br = new BufferedReader(new InputStreamReader(exec.getInputStream()));
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println(command + " --> "+ sb.toString());

    }
}

```

`javac NodeCheck.java`然后`java NodeCheck`可以看到输出——

```
[root@testme]# javac nodecheck.java 
[root@testme]# java NodeCheck 
node -v --> v10.13.0

/usr/bin/node -v --> v6.17.1

/root/.nvm/versions/node/v10.13.0/bin/node -v --> v10.13.0

```

这样可以确定，Java应用中使用了`v6.17.1`版本的node，所以会提示`Unexpected token ...`

简单暴力解决，动态判断两个node版本，不一致时直接使用新版本覆盖`/usr/bin`目录下的node。
