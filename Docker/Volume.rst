============
使用Volumes
============
Volumes是保存Docker容器产生和使用数据的首选机制。bind mounts依赖于主机的目录结构，而Volumes完全由Docker管理。与bind mounts相比，Volumes有几个优势：

* Volumes 比 bind mounts 更容易备份或迁移。
* 你能够用 Docker CLI 命令或 Docker API 管理 volumes。
* Volumes可以在Linux和Windows容器上工作。
* Volumes可以在多个容器之间更安全地共享。 
* Volumes驱动程序允许您在远程主机或云提供者上存储卷，加密卷的内容，或添加其他功能。
* 新Volumes可以通过容器预先填充其内容。

