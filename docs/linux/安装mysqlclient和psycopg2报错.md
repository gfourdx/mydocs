# 安装mysqlclient和psycopg2报错

---

使用pip安装上述两个包需要解决以下的依赖问题, 具体来说就是:

- pkg-config
- gcc 或 build-essential
- python3-dev
-  libmariadb-dev 或 libmariadb-dev-compat 或 default-libmysqlclient-dev
- libpq-dev

---

`build-essential`不仅包括GCC编译器, 还包括其他构建软件所需的工具和库, 如GNU C++编译器(g++)、make、libc-dev等, 它提供了一组广泛的工具，方便开发人员进行软件开发和构建

在打包Docker环境时为减少体积可只安装gcc, 不安装build-essential

---

**libmariadb-dev**：

是专门为MariaDB数据库提供的开发包, 它包含了用于构建和连接与MariaDB相关的应用程序所需的头文件和库, 专注于支持MariaDB的特定功能和特性, 并提供了与MariaDB数据库服务器密切集成的开发工具

**libmariadb-dev-compat**：

是为了兼容MySQL而提供的包, 它提供了与MySQL开发包类似的头文件和库，以便在使用MariaDB时能够编译和连接基于MySQL的应用程序, 这个软件包可以确保现有的MySQL应用程序可以在使用MariaDB数据库时无需进行大量修改而继续工作

即如果你已经有一个基于MySQL的应用程序, 并且想要将其迁移到MariaDB而不需要进行大量修改, 那么可以考虑安装改包以保持向后兼容性

可以简单的理解为在功能上`libmariadb-dev-compat`包含 `libmariadb-dev`同时与MySQL兼容

使用apt安装该包的同时也会安装`libmariadb-dev`

**default-libmysqlclient-dev**：是用于默认的MySQL数据库服务器(通常是Oracle MySQL)的包, 包含了构建和连接MySQL客户端应用程序所需的头文件和库, 如果你使用的是Oracle MySQL数据库, 或者在Debian系发行版中安装了默认的MySQL服务器(MariaDB)，那么可以使用该包

使用apt安装该包的同时也会安装`libmariadb-dev-compat`和`libmariadb-dev`

总结一下, 在Debian系发行版上：

- 如果你正在使用MariaDB, 并且需要开发与MariaDB直接交互的应用程序, 则可以安装`libmariadb-dev`或`ibmariadb-dev-compat`或`default-libmysqlclient-dev`
- 如果你正在使用MariaDB, 并且想要编写与MySQL兼容的应用程序, 或迁移到MySQL, 则需要安装`ibmariadb-dev-compat`
- 如果你正在使用MariaDB, 并且想要编写与Oracle MySQL兼容的应用程序或迁移到Oracle MySQL, 则需要安装`default-libmysqlclient-dev`

简单说:

MariaDB -> libmariadb-dev

MySQL -> ibmariadb-dev-compat

Oracle MySQL -> default-libmysqlclient-dev

或者直接安装default-libmysqlclient-dev , 不再考虑数据库到底是MariaDB还是MySQL还是Oracle MySQL