# Nom&yarn

## npm install流程

![068739612.png](https://gitee.com/myreally/pic/raw/master/Cip5yF_axkqAclTFAAJmlxGYSmI551.png)

npm install执行后，会优先检查并获取npm配置，**项目级的npmrc文件 > 用户级的.npmrc文件 > 全局级的.npmrc文件 > npm内部的.npmrc文件**

然后检查是否有package-lock.json文件，如果有，则检查package-lock.json和 package.json依赖是否一致；

一致：直接使用package-lock.json中的信息，从缓存或网络资源中加载依赖

不一致：使用npm版本进行处理

如果没有package-lock.json文件，则会根据package.json递归构建依赖树，然后按照构建好的依赖树下载完整的依赖资源，在下载的时候会检查是否存在相关的资源缓存

存在：则将缓存内容解压到node

## Npm 缓存机制

```shell
// 执行如下命令
npm config get cache
```

通过上述命令可以得到缓存存储位置 **/Users/songjun/.npm**(Mac OS中，npm默认)

![image-20211017152033291](https://gitee.com/myreally/pic/raw/master/image-20211017152033291.png)

![image-20211017154443810](https://gitee.com/myreally/pic/raw/master/image-20211017154443810.png)

```shell
// 清除缓存
npm cache clean --force
```

**这些缓存如何被储存并被利用的呢？**

这就和 npm install 机制联系在了一起。当 npm install 执行时，通过pacote把相应的包解压在对应的 node_modules 下面。npm 在下载依赖时，先下载到缓存当中，再解压到项目 node_modules 下。pacote 依赖npm-registry-fetch来下载包，npm-registry-fetch 可以通过设置 cache 属性，在给定的路径下根据IETF RFC 7234生成缓存数据。

接着，在每次安装资源时，根据 package-lock.json 中存储的 integrity、version、name 信息生成一个唯一的 key，这个 key 能够对应到 index-v5 目录下的缓存记录。如果发现有缓存资源，就会找到 tar 包的 hash，根据 hash 再去找缓存的 tar 包，并再次通过pacote把对应的二进制文件解压到相应的项目 node_modules 下面，省去了网络下载资源的开销。

## npm link

npm link 的本质就是软链接，它主要做了两件事：

为目标 npm 模块（npm-package-1）创建软链接，将其链接到全局 node 模块安装路径 /usr/local/lib/node_modules/ 中；

为目标 npm 模块（npm-package-1）的可执行 bin 文件创建软链接，将其链接到全局 node 命令安装路径 /usr/local/bin/ 中。

## npx的作用

可以直接执行 node_modules/.bin 文件夹下的文件。在运行命令时，npx 可以自动去 node_modules/.bin 路径和环境变量 $PATH 里面检查命令是否存在，而不需要再在 package.json 中定义相关的 script。

npx 另一个更实用的好处是：npx 执行模块时会优先安装依赖，但是在安装执行后便删除此依赖，这就避免了全局安装模块带来的问题。

## yarn特点

**确定性：**通过 yarn.lock 等机制，保证了确定性。即不管安装顺序如何，相同的依赖关系在任何机器和环境下，都可以以相同的方式被安装。（在 npm v5 之前，没有 package-lock.json 机制，只有默认并不会使用的npm-shrinkwrap.json。）

**采用模块扁平安装模式：**将依赖包的不同版本，按照一定策略，归结为单个版本，以避免创建多个副本造成冗余（npm 目前也有相同的优化）。

**网络性能更好：**Yarn 采用了请求排队的理念，类似并发连接池，能够更好地利用网络资源；同时引入了更好的安装失败时的重试机制。

**采用缓存机制**，实现了离线模式（npm 目前也有类似实现）。

**Yarn 默认使用 prefer-online 模式，即优先使用网络数据。如果网络数据请求失败，再去请求缓存数据。**

检测（checking）→ 解析包（Resolving Packages） → 获取包（Fetching Packages）→ 链接包（Linking Packages）→ 构建包（Building Packages）

![图片14.png](https://gitee.com/myreally/pic/raw/master/CgqCHl_ZflCANVu8AAJJZZYzwhs026.png)

Yarn 安装流程图

检测包（checking）

这一步主要是检测项目中是否存在一些 npm 相关文件，比如 package-lock.json 等。如果有，会提示用户注意：这些文件的存在可能会导致冲突。在这一步骤中，也会检查系统 OS、CPU 等信息。

解析包（Resolving Packages）

这一步会解析依赖树中每一个包的版本信息。

首先获取当前项目中 package.json 定义的 dependencies、devDependencies、optionalDependencies 的内容，这属于首层依赖。

接着采用遍历首层依赖的方式获取依赖包的版本信息，以及递归查找每个依赖下嵌套依赖的版本信息，并将解析过和正在解析的包用一个 Set 数据结构来存储，这样就能保证同一个版本范围内的包不会被重复解析。

对于没有解析过的包 A，首次尝试从 yarn.lock 中获取到版本信息，并标记为已解析；

如果在 yarn.lock 中没有找到包 A，则向 Registry 发起请求获取满足版本范围的已知最高版本的包信息，获取后将当前包标记为已解析。

总之，在经过解析包这一步之后，我们就确定了所有依赖的具体版本信息以及下载地址。

![Drawing 2.png](https://gitee.com/myreally/pic/raw/master/CgqCHl_TbimACnDOAAFMC14gP8I289.png)

解析包获取流程图

获取包（Fetching Packages）

这一步我们首先需要检查缓存中是否存在当前的依赖包，同时将缓存中不存在的依赖包下载到缓存目录。说起来简单，但是还是有些问题值得思考。

比如：如何判断缓存中是否存在当前的依赖包？其实 Yarn 会根据 cacheFolder+slug+node_modules+pkg.name 生成一个 path，判断系统中是否存在该 path，如果存在证明已经有缓存，不用重新下载。这个 path 也就是依赖包缓存的具体路径。

对于没有命中缓存的包，Yarn 会维护一个 fetch 队列，按照规则进行网络请求。如果下载包地址是一个 file 协议，或者是相对路径，就说明其指向一个本地目录，此时调用 Fetch From Local 从离线缓存中获取包；否则调用 Fetch From External 获取包。最终获取结果使用 fs.createWriteStream 写入到缓存目录下。

![Drawing 3.png](https://gitee.com/myreally/pic/raw/master/Ciqc1F_TbjKAThkOAAEsp0sOHUc622.png)

获取包流程图

链接包（Linking Packages）

上一步是将依赖下载到缓存目录，这一步是将项目中的依赖复制到项目 node_modules 下，同时遵循扁平化原则。在复制依赖前，Yarn 会先解析 peerDependencies，如果找不到符合 peerDependencies 的包，则进行 warning 提示，并最终拷贝依赖到项目中。

这里提到的扁平化原则是核心原则，我也会在后面内容进行详细的讲解。

![Drawing 4.png](https://gitee.com/myreally/pic/raw/master/Ciqc1F_Tbj2AWiPOAADyaZB-wGw502.png)

链接包解析流程图

构建包（Building Packages）

如果依赖包中存在二进制包需要进行编译，会在这一步进行。

了解了 npm 和 Yarn 的安装原理还不是“终点”，因为一个应用项目的依赖错综复杂。接下来我将从“依赖地狱”说起，帮助你加深对依赖机制相关内容的理解，以便在开发生产中灵活运用。

破解依赖管理困境
早期 npm（npm v2）的设计非常简单，在安装依赖时将依赖放到项目的 node_modules 文件中；同时如果某个直接依赖 A 还依赖其他模块 B，作为间接依赖，模块 B 将会被下载到 A 的 node_modules 文件夹中，依此递归执行，最终形成了一颗巨大的依赖模块树。

这样的 node_modules 结构，的确简单明了、符合预期，但对大型项目在某些方面却不友好，比如可能有很多重复的依赖包，而且会形成“嵌套地狱”。

那么如何理解“嵌套地狱”呢？

项目依赖树的层级非常深，不利于调试和排查问题；

依赖树的不同分支里，可能存在同样版本的相同依赖。比如直接依赖 A 和 B，但 A 和 B 都依赖相同版本的模块 C，那么 C 会重复出现在 A 和 B 依赖的 node_modules 中。

这种重复问题使得安装结果浪费了较大的空间资源，也使得安装过程过慢，甚至会因为目录层级太深导致文件路径太长，最终在 Windows 系统下删除 node_modules 文件夹出现失败情况。

因此 npm v3 之后，node_modules 的结构改成了扁平结构，按照上面的例子（项目直接依赖模块 A，A 还依赖其他模块 B），我们得到下面的图示：

![图片10.png](https://gitee.com/myreally/pic/raw/master/Ciqc1F_ZfoKACluCAADJ2SvodGg411.png)

npm 不同版本的安装结构图 ①

当项目新添加了 C 依赖，而它依赖另一个版本的 B v2.0。这时候版本要求不一致导致冲突，B v2.0 没办法放在项目平铺目录下的 node_moduls 文件当中，npm v3 会把 C 依赖的 B v2.0 安装在 C 的 node_modules 下：

![图片9.png](https://gitee.com/myreally/pic/raw/master/Ciqc1F_Zf1eAWVhcAADO_4H0sjA082.png)

npm 不同版本的安装结构图 ②

接下来，在 npm v3 中，假如我们的 App 现在还需要依赖一个 D，而 D 也依赖 B v2.0 ，我们会得到如下结构：

![图片17.png](https://gitee.com/myreally/pic/raw/master/Cip5yF_Zf2mABSwEAAC-YH5jkcQ965.png)

npm 安装结构图 ①

这里我想请你思考一个问题：为什么 B v1.0 出现在项目顶层 node_modules，而不是 B v2.0 出现在 node_modules 顶层呢？

其实这取决于模块 A 和 C 的安装顺序。因为 A 先安装，所以 A 的依赖 B v1.0 率先被安装在顶层 node_modules 中，接着 C 和 D 依次被安装，C 和 D 的依赖 B v2.0 就不得不安装在 C 和 D 的 node_modules 当中了。因此，模块的安装顺序可能影响 node_modules 内的文件结构。

我们继续依赖工程化之旅。假设这时候项目又添加了一个依赖 E ，E 依赖了 B v1.0 ，安装 E 之后，我们会得到这样一个结构：

![图片6.png](https://gitee.com/myreally/pic/raw/master/CgpVE1_Zf4aADdDJAADNnUsWnlc423.png)

npm 安装结构图 ②

此时对应的 package.json 中，依赖包的顺序如下：

```javascript
{
    A: "1.0",
    C: "1.0",
    D: "1.0",
    E: "1.0"
}
```

如果我们想更新模块 A 为 v2.0，而模块 A v2.0 依赖了 B v2.0，npm v3 会怎么处理呢？

整个过程是这样的：

删除 A v1.0；

安装 A v2.0；

留下 B v1.0 ，因为 E v1.0 还在依赖；

把 B v2.0 安装在 A v2.0 下，因为顶层已经有了一个 B v1.0。

它的结构如下：

![图片5.png](https://gitee.com/myreally/pic/raw/master/Cip5yF_Zf6iAClRIAADSW-XFvzA495.png)

npm 安装结构图 ③

这时模块 B v2.0 分别出现在了 A、C、D 模块下——重复存在了。

通过这一系列操作我们可以看到：npm 包的安装顺序对于依赖树的影响很大。模块安装顺序可能影响 node_modules 内的文件数量。

这里一个更理想的依赖结构理应是：

![图片4.png](https://gitee.com/myreally/pic/raw/master/CgpVE1_Zf76ADk3NAADVWLHAHTo908.png)

npm 安装结构图 ④

过了一段时间，模块 E v2.0 发布了，并且 E v2.0 也依赖了模块 B v2.0 ，npm v3 更新 E 时会怎么做呢？

删除 E v1.0；

安装 E v2.0；

删除 B v1.0；

安装 B v2.0 在顶层 node_modules 中，因为现在顶层没有任何版本的 B 了。

此时得到图：

![图片3.png](https://gitee.com/myreally/pic/raw/master/Ciqc1F_Zf82Abr7LAADYStAX7VU318.png)

npm 安装结构图 ⑤

这时候，你可以明显看到出现了较多重复的依赖模块 B v2.0。我们可以删除 node_modules，重新安装，利用 npm 的依赖分析能力，得到一个更清爽的结构。

实际上，更优雅的方式是使用 npm dedupe 命令，得到：

![图片2.png](https://gitee.com/myreally/pic/raw/master/CgqCHl_Zf-WAb-BnAADAe1GD0YY021.png)

npm 安装结构图 ⑥

实际上，Yarn 在安装依赖时会自动执行 dedupe 命令。整个优化的安装过程，就是上一讲提到的扁平化安装模式，也是需要你掌握的关键内容。

