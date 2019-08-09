==================
Dockerfile文件详解
==================
官网文档地址: https://docs.docker.com/v17.09/engine/reference/builder/#environment-replacement

Docker可以通过读取Dockerfile中的指令自动构建镜像。 Dockerfile是一个文本文档，其中包含用户可以在命令行上调用以装配镜像的所有命令。 使用docker build用户可以创建一个连续执行多个命令行指令的自动构建。

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

docker build命令从Dockerfile和上下文构建一个映像。 构建的 **上下文** 是指“指定位置PATH或URL中的文件集”。 PATH是本地文件系统上的一个目录。 URL是一个Git存储库位置。

上下文是递归处理的。因此， PATH包含任何子目录，且URL包含存储库及其子模块。

在docker CLI将上下文发送到docker守护进程之前，它会在上下文的根目录中查找名为.dockerignore的文件。如果此文件存在，CLI将修改上下文以排除与其中的模式匹配的文件和目录。这有助于避免不必要地将大型或敏感文件和目录发送到守护进程，并可能使用ADD或COPY将它们添加到镜像。

 以＃开头，则此行被视为注释。
 
 可以使用? * **通配符
 
 !（感叹号）开头的行可用于排除例外情况

Dockerfile位于上下文的根目录中。 你可以将-f标志与docker build一起使用，以指向文件系统中任何位置的Dockerfile。

该指令不区分大小写。 但是，惯例是让它们成为大写的，以便更容易地将它们与参数区分开来。

Docker按顺序在Dockerfile中运行指令。 Dockerfile必须以FROM指令开头。 

指令的一般格式为：INSTRUCTION arguments

LABEL
+++++
  LABEL指令将元数据添加到镜像中。LABEL是一个键值对。要在LABEL值中包含空格，请使用引号和反斜杠。

  镜像中可以有多个标签。要指定多个标签，Docker建议LABEL在可能的情况下将标签组合到单个指令中。每个LABEL指令产生一个新的层。
::

  LABEL multi.label1="value1" multi.label2="value2" other="value3"
  或
  LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"

ARG
++++

FROM
++++
 指定基础镜像，必须为第一个命令。
* FROM <image> [AS <name>
* FROM <image>[:<tag>] [AS <name>]
* FROM <image>[@<digest>] [AS <name>]

FROM指令初始化新的构建阶段并为后续指令设置基础镜像。因此，有效的Dockerfile必须以FROM指令开头。镜像可以是任何有效镜像 - 通过从公共存储库中提取镜像（pulling an image）来启动它尤其容易。

ARG是Dockerfile中唯一可以在FROM之前的指令。
在FROM之前声明的ARG在构建阶段之外，因此在FROM之后的任何指令中都不能使用它。 要使用在第一个FROM之前声明的ARG变量的缺省值，请在构建阶段内使用没有值的ARG指令
::
  ARG VERSION=latest
  FROM busybox:$VERSION
  ARG VERSION
  RUN echo $VERSION > image_version

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

<dest>是绝对路径，或相对于WORKDIR的路径，<src>将在目标容器中复制到该路径中。
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

ADD
++++
 同COPY
* ADD [--chown=<user>:<group>] <src>... <dest>
* ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]

与COPY的不同点
 * <src>可以是URLs
 * 如果<src>是可识别的压缩格式（identity、gzip、bzip2或xz）的本地tar存档，则将其解压缩为目录。远程URL中的资源不解压缩。

VOLUME
+++++++

RUN
++++
  构建镜像时执行的命令，复杂的RUN请用反斜线\换行，避免无用分层，合并多条命令成一行！
* RUN <command> (shell形式，命令在shell中运行，Linux默认为/bin/sh -c, Windows为cmd /S /C)
* RUN ["executable", "param1", "param2"] (exec形式)
 **exec形式** 不会调用command shell。这意味着不会发生正常的shell处理。例如，RUN [ "echo", "$HOME" ]不会对$HOME执行变量替换。如果你想要shell处理，那么要么使用shell形式，要么直接执行shell.

CMD
+++
  设置容器启动后默认执行的命令和参数
* CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；
* CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用；
* CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；

如果docker run指定了其他命令，CMD命令被忽略。

如果定义了多个CMD，只有最后一个会执行。

命令执行的形式
 * Shell 形式
   启用新的sub-shell（新的子进程）,然后在其下执行命令，可以使用环境变量(这是Shell的特性)。
 * Exec 形式 
   使用exec命令并不启动新的shell，而是使用执行命令替换当前的shell进程(此命令进程的pid为1)，并且将老进程的环境清理掉，而且exec命令后的其他命令将不再执行。不可以使用环境变量。

ENTRTYPOINT
+++++++++++
  设置容器启动时运行的命令，允许你配置将作为可执行文件运行的容器。不可被docker run 提供的参数覆盖，一定会执行。

* ENTRYPOINT ["executable", "param1", "param2"] (exec形式, 首选)
* ENTRYPOINT command param1 param2 (shell形式)

docker run <image>的命令行参数将附加在exec形式的ENTRYPOINT的所有元素后面，并覆盖使用CMD指定的所有元素。这允许将参数传递给ENTRYPOINT，即docker run <image> -d将-d参数传递给ENTRYPOINT。你可以使用docker run --entrypoint标志覆盖ENTRYPOINT指令。

shell形式防止使用任何CMD命令参数或run命令行参数，缺点是ENTRYPOINT将作为/bin/sh-c的子命令启动，而该子命令不传递信号。这意味着可执行文件将不能成为容器的pid 1，也不会接收unix信号，因此可执行文件将不会从docker stop<container>接收SIGTERM信号。

每个Dockerfile中只能有一个ENTRYPOINT，当指定多个时，只有最后一个有效。

你可以使用ENTRYPOINT的exec形式设置比较稳定的默认命令和参数，然后使用任一形式的CMD来设置很可能更改的其他默认值。

Exec形式 ENTRYPOINT 示例
-------------------------
Dockerfile内容：
::
  FROM ubuntu
  ENTRYPOINT ["top", "-b"]
  CMD ["-c"]

启动容器，可以看到top是唯一的进程：
::

& docker build -t top .
& docker run -it --rm --name test top -H

要进一步检查结果
::
  docker exec -it test ps aux

同时可以优雅地请求使用docker stop test来关闭top。

Shell形式 ENTRYPOINT 示例
-------------------------
可以为ENTRYPOINT指定一个纯字符串，它将在/bin/sh -c中执行。这种形式将使用shell处理来替换shell环境变量，并将忽略任何CMD或docker run命令行参数。要确保docker stop能正确发信号给任何长时间运行的ENTRYPOINT可执行文件，你需要记住用exec启动它。

Dockerfile内容：
::
  FROM ubuntu
  ENTRYPOINT exec top -b

启动容器，可以看到单个PID 1进程：
::

& docker build -t top .
& docker run -it --rm --name test top -H

它将在docker stop时干净地退出：

如果忘记将exec添加到ENTRYPOINT的开头，启动容器，可以看到指定的ENTRYPOINT不是PID 1。如果运行docker stop test，容器将不会干净地退出stop命令将被强制在超时后发送SIGKILL。

了解CMD和ENTRYPOINT如何相互作用：
-------------------------------
CMD和ENTRYPOINT指令都定义了运行容器时执行的命令。 这里有点规则描述他们之间的合作。

* Dockerfile应至少指定一个CMD或ENTRYPOINT命令。
* 使用容器作为可执行文件时，应定义ENTRYPOINT。
* CMD应该用作为ENTRYPOINT命令定义默认参数或在容器中执行特定命令的方法。
* 当使用可选参数运行容器时，将覆盖CMD。

EXPOSE
+++++++
VOLUME
++++++
  创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。
* VOLUME ["/data"]。

USER
+++++
  指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。
* USER daemon。
当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：RUN groupadd -r postgres && useradd -r -g postgres postgres。要临时获取管理员权限可以使用 gosu，而不推荐 sudo。