
文档官网: https://hub.docker.com/_/mariadb

The startup configuration is specified in the file /etc/mysql/my.cnf, and that file in turn includes any files found in the /etc/mysql/conf.d directory that end with .cnf. Settings in files in this directory will augment and/or override settings in /etc/mysql/my.cnf. If you want to use a customized MySQL configuration, you can create your alternative configuration file in a directory on the host machine and then mount that directory location as /etc/mysql/conf.d inside the mariadb container.