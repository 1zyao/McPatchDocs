# 注意的事项

## 文件的下载位置

文件的实际更新位置，不是直接下载到客户端程序旁边的，而是自动搜索.minecraft的父目录开始的。因此到处移动客户端程序不会影响更新结果。如果要禁用这个机制，可以在配置文件里调整`base-path`选项（一般不建议改这个选项）

如果要移动客户端程序，记得带着`mc-patch-version.txt`和配置文件一起移动，否则程序会找不到配置文件

## 文件会更新失败

若某个文件在更新时，检测到文件内容被人为修改过了（或删除），那么这个文件会跳过更新（丢失更新状态），后续所有对这个文件的更新都会直接跳过，也就是会更新失败

这不是程序BUG，这是特意的设计。原因是因为管理端打包会对比新旧文件，生成差异文件（补丁文件）

+ 旧文件 + 新文件 => 补丁文件

然后客户端会下载这个补丁文件把旧文件修补（合并）成新文件

+ 补丁文件 + 旧文件 => 新文件

这样只记录差异的方法可以很大程度上节省更新流量

但这个方法有一个缺点，客户端合并文件时，如果旧文件被修改过，那么最终合并出来的新文件数据就是完全错乱的

因此客户端程序设计了这个跳过更新的机制，避免文件数据错乱，同时利用这个机制，可以保留玩家自己的个性化设置数据不被更新（直到这个被修改的文件在服务端被删除才会打破循环，重新加入更新）

这个机制导致了玩家不能“手贱”修改文件，否则会导致这个文件从此之后的更新全部失败

## 版本发错了怎么办

版本号一旦发布就不能撤回，撤回可能会导致客户端某些文件更新永久更新失败，而且这种问题很难发现和调试。你应该额外再发布一个版本来替代撤回

如果你100%确定刚发布的错误版本没有任何人下载的话，可以使用以下方法来撤回：

1. 打开`public`目录下的版本列表文件：`mc-patch-versions.txt`或`versions.txt`，将错误版本那一行连带后面所有的行都删除掉（**这一行前面部分的千万别改动**）
2. 比如123456这6个版本中，4出了问题，就要撤回456三个版本，就在版本列表文件里删除456这三行，使3这一行成为文件末尾
3. 接着删除public目录下456这三个版本对应的json文件和bin文件
4. 最后运行管理端，在主菜单输入（1.0版本输入`bv`，1.1版本输入`revert`）进入还原菜单，恢复workspace目录和history目录的内容。恢复所需的时间和已有版本的数量成正比，如果版本非常多，过程可能会非常慢
5. 这样就回退到了你发布错误版本号之前的状态了
6. 如果你不能100%保证没有任何人下载过这个错误的版本，就不要撤回版本，否则那个人会出现各种各样的奇奇怪怪的问题

## 怎么删掉管理端不存在的文件

在使用过程中，有时会想删除一些在客户端存在，但管理端不存在的文件。这些文件没法用往常办法删除

删掉这些不存在的文件有两个方法，一个是加入更新之后再删掉，一个是直接修改更新包内部数据

通常情况下建议使用第一个方法，更加安全可靠不易出错。第二种方法适合大佬，因为要改一些内部数据，但好处是快，能一步到位

第一种方法很简单

1. 客户端有个叫`abc.jar`的文件需要删除
2. 先在工作空间目录下的相同位置下，创建一个同名的空文件`abc.jar`
3. 然后开启管理端创建一个新版本（这样`abc.jar`就从未加入更新的状态变为加入更新的状态了）
4. 接着把刚创建的`abc.jar`删掉，再创建一个新版本
5. 客户端会依次下载这两个版本，最终的效果就是这个文件被成功删除掉了

第二种方法仅适用于管理端1.1或者更新的版本

1. 创建一个空的版本号
2. 直接打开更新包zip里的`.mcpatch-meta.json`文件
3. 在`old-files`字段处添加要删除的文件的相对路径（相对工作空间目录的路径），路径分隔符用正斜线
4. 保存关闭`.mcpatch-meta.json`文件并更新回更新包zip文件

## 不小心修改了history目录

如果你不小心修改了history目录下的内容，可以在主菜单输入（1.0版本输入`bv`，1.1版本输入`revert`）进入还原菜单来同时还原workspace目录和history目录，注意使用此命令会丢失已有修改，注意备份重要数据

恢复所需的时间和已有版本的数量成正比，如果版本非常多，过程可能会非常慢

## 加密配置文件和版本号文件

> 此特性仅McPatchClient 1.0.11或更高版本支持

这里的加密不是真的加密，而是将内容以base64编码后以非明文保存，不能起到绝对的安全防护的作用

加密配置文件（config.yml/mc-patch-config.yml）：将整个配置文件内容复制后使用Base64进行编码，然后删掉原有内容，将编码后的内容粘贴进去，然后在文件的开头添加单个英文冒号`:`用来告诉程序配置文件被“加密”了，使用之前要先“解密”

加密版本号文件（mc-patch-version.txt）：将整个版本号文件内容复制后使用Base64进行编码，然后删掉原有内容，将编码后的内容粘贴进去，然后在文件的开头添加单个英文冒号`:`用来告诉程序配置文件被“加密”了，使用之前要先“解密”

注意：配置文件和版本号文件“加密”后，不会影响日志文件里的账号密码信息的显示，如果在意，可以在配置文件里禁用日志文件生成

## 避免仅修改文件名大小写

如果有个文件叫abc.jar你将其改名成Abc.jar然后打包，这种情况就会出问题！

你改成Abc1.jar都没事，改成Ab.jar也可以，但唯独Abc.jar不行

客户端遇到这种文件时，会直接删掉abc.jar，而不下载Abc.jar，然后这个文件就会永久更新失败

这个问题的原因是文件系统不区分文件名大小写造成的

从管理端1.1.9版本开始遇到这种文件会直接打包失败，避免将有问题的数据带进更新包。而在这之前的版本中，需要人为避免出现这种情况

## 版本号并非判断新旧的依据

版本号只是一个普通的标签，是给人类看的，程序不会解析对比版本号的文字，也不作为任何判断版本前后的依据。

实际版本的新旧顺序是按你打包的时间来判断的，后打的版本总是比先打的版本要新

版本号并没有规定一定要往高走，也可以往低走。就是说你可以从1.5版本更新到1.4版本。这算更新而非回退，因为1.4版本晚于1.5版本打包，所以被认为是较新的版本

至于哪个版本更新，哪个版本更旧？可以亲自打开版本列表文件看看（**注意只能看不能改**）

## 客户端配置文件可以隐藏吗

不行！不仅是配置文件，连日志文件也不能隐藏。因为隐藏的文件没有写入权限，程序会报错

## 修复客户端的文件

使用此功能需要1.1版本或者更高版本的管理端和服务端

从1.1版本开始，更新包开始划分为增量包和全量包两种。最先创建的第一个版本永远是全量包，后面创建的永远是增量包。无论客户端文件损坏成什么样，只要能运行McPatch就可以利用全量更新包机制把整个客户端重新安装一次

方法很简单：删除客户端的版本号文件即可。客户端会自动重头开始下载所有的更新包并依次安装

需要注意：

1. 这个功能如果在配置文件里被手动关闭了，是不起作用的（默认是开启的）
2. 完整下载一边所有的更新包会非常消耗时间，也非常消耗流量和带宽
3. 文件的修复范围仅限于加入过更新的文件（也就是工作空间目录里曾经出现的文件和目录）

