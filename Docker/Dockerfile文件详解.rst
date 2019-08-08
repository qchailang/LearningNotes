==================
Dockerfile文件详解
==================
Docker可以通过读取Dockerfile中的指令自动构建镜像。 Dockerfile是一个文本文档，其中包含用户可以在命令行上调用以装配镜像的所有命令。 使用docker build用户可以创建一个连续执行多个命令行指令的自动构建。

docker build命令从Dockerfile和上下文构建一个映像。 **构建的上下文** 是指“指定位置PATH或URL中的文件集”。 PATH是本地文件系统上的一个目录。 该URL是一个Git存储库位置。

上下文是递归处理的。因此， PATH包含任何子目录，且URL包含存储库及其子模块。

在docker CLI将上下文发送到docker守护进程之前，它会在上下文的根目录中查找名为.dockerignore的文件。如果此文件存在，CLI将修改上下文以排除与其中的模式匹配的文件和目录。这有助于避免不必要地将大型或敏感文件和目录发送到守护进程，并可能使用ADD或COPY将它们添加到镜像。

 以＃开头，则此行被视为注释。
 
 可以使用? * **通配符
 
 !（感叹号）开头的行可用于排除例外情况


Dockerfile位于上下文的根目录中。 你可以将-f标志与docker build一起使用，以指向文件系统中任何位置的Dockerfile。

该指令不区分大小写。 但是，惯例是让它们成为大写的，以便更容易地将它们与参数区分开来。

Docker按顺序在Dockerfile中运行指令。 Dockerfile必须以FROM指令开头。 

常用指令
-------
LABEL
+++++
  LABEL指令将元数据添加到image中。LABEL是一个键值对。要在LABEL值中包含空格，请使用引号和反斜杠。

  image中可以有多个标签。要指定多个标签，Docker建议LABEL在可能的情况下将标签组合到单个指令中。每个LABEL指令产生一个新的层。

::

  LABEL multi.label1="value1" multi.label2="value2" other="value3"
  或
  LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
FROM
++++
 指定基础镜像，必须为第一个命令。
* FROM <image> [AS <name>
* FROM <image>[:<tag>] [AS <name>]
* FROM <image>[@<digest>] [AS <name>]

FROM指令初始化新的构建阶段并为后续指令设置基础镜像。因此，有效的Dockerfile必须以FROM指令开头。镜像可以是任何有效镜像 - 通过从公共存储库中提取镜像（pulling an image）来启动它尤其容易。

ARG是Dockerfile中唯一可以在FROM之前的指令。

FROM可以在单个Dockerfile中多次出现以创建多个镜像，或者使用一个构建阶段作为另一个构建阶段的依赖项。只需在每个新的FROM指令之前记下提交输出的最后一个图像ID。每个FROM指令清除先前指令创建的任何状态。

通过将AS name添加到FROM指令，可以将可选的名称赋予新的构建阶段。该名称可以在后续的FROM和COPY --from=<name|index>指令中使用，以引用此阶段构建的镜像。

tag（标记）或digest（docker image ls 命令显示的 IMAGE ID 的值）值是可选的。如果省略其中任何一个，则构建器默认采用latest（最新）标记。如果找不到tag值，构建器将返回错误。

COPY
++++
 将本地文件添加到容器中。
* COPY [--chown=<user>:<group>] <src>... <dest>
* COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]（路径中包含空格时使用这种格式）

COPY指令从<src>复制新文件或目录，并将它们添加到容器的文件系统的<dest>路径中。

可以指定多个<src>资源，但文件和目录的路径将被解释为相对于构建上下文的源。

每个<src>可能包含通配符，例如
::
  COPY hom* /mydir/        # adds all files starting with "hom"
  COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

<dest>是绝对路径，或相对于WORKDIR的路径，源将在目标容器中复制到该路径中。
::
  COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
  COPY test /absoluteDir/  # adds "test" to /absoluteDir/

除非可选的--chown标志指定给定用户名、组名或UID/GID组合以请求添加内容的特定所有权，否则将使用UID和GID为0创建所有新文件和目录。 --chown标志的格式允许用户名和组名字符串或任意组合的直接整数UID和GID。 提供没有组名的用户名或没有GID的UID将使用与GID相同的数字UID。 如果提供了用户名或组名，则容器的根文件系统/etc/passwd和/etc/group文件将分别用于执行从名称到整数UID或GID的转换。 以下示例显示了--chown标志的有效定义:
::
  COPY --chown=55:mygroup files* /somedir/
  COPY --chown=bin files* /somedir/
  COPY --chown=1 files* /somedir/
  COPY --chown=10:11 files* /somedir/

如果容器根文件系统不包含/etc/passwd或/etc/group文件，并且在--chown标志中使用了用户名或组名，则构建将在COPY操作上失败。使用数字ID不需要查找，也不依赖于容器根文件系统内容。

COPY遵守以下规则：

* <src>路径必须位于构建的上下文中；你不能COPY ../something/something，因为docker build的第一步是将上下文目录（和子目录）发送到docker守护进程。

* 如果<src>是目录，则复制目录的全部内容，包括文件系统元数据。

   注意：不复制目录本身，只复制其内容。

* 如果<src>是任何其他类型的文件，则将其与元数据一起单独复制。在这种情况下，如果<dest>以尾部斜杠/结束，则将其视为目录，<src>的内容将写入<dest>/base(<src>)。

* 如果直接或由于使用通配符指定了多个<src>资源，则<dest>必须是目录，并且必须以斜杠/结尾。

* 如果<dest>不以尾部斜杠结束，则它将被视为常规文件，<src>的内容将写入<dest>。

* 如果<dest>不存在，则会在其路径中创建所有缺少的目录。

RUN
++++
  构建镜像时执行的命令，复杂的RUN请用反斜线换行，避免无用分层，合并多条命令成一行！
* RUN <command> (shell形式，命令在shell中运行，Linux默认为/bin/sh -c, Windows为cmd /S /C)
* RUN ["executable", "param1", "param2"] (exec form)

CMD
+++
  设置容器启动后默认执行的命令和参数
* Shell
* Exec 格式
容器启动时默认执行的命令

如果docker run 指定了其他命令，CMD命令被忽略

如就是定义了多个CMD，只有最后一个会执行

ENTRTYPOINT
+++++++++++
  设置容器启动时运行的命令，让容器以应用程序或者服务的形式运行，不会被忽略，一定会执行
