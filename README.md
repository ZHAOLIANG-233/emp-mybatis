# emp-mybatis

《Java 框架技术（一）》实验一：**纯 Java + MyBatis / MyBatis-Plus 环境搭建与 CRUD 实操**。

单模块 Maven 工程，不使用 Spring、不使用任何 starter：SqlSessionFactory、数据源、
拦截器与 Mapper 注册全部在代码里手工装配。

- 作者：赵亮　学号：20241850　班级：4 班
- 环境：macOS 26.3.1 / OpenJDK 17.0.19 / Maven 3.10.0 / MySQL 9.6.0
- 规模：15 个测试类 / **116 个测试用例**，全部通过

---

## 一、在 IDEA 里跑起来

### 准备数据库（两个表）

```bash
# 建库建表 + 导入数据
mysql -u root -p < sql/01_schema.sql     # 建库 ssm_emp + 两张表
mysql -u root -p < sql/02_seed.sql       # 6 个部门 + 16 名员工
mysql -u root -p < sql/03_verify.sql     # 核对：16 / 15 / 1 / 12 / 3
```

建出来的是两张表：

| 表 | 列数 | 说明 |
| --- | --- | --- |
| `emp` | 17 | 员工表：工号、姓名、性别、出生日期、邮箱、电话、部门、岗位、薪资、入职日期、状态、逻辑删除标记、版本号等 |
| `sys_dept` | 8 | 部门维表：编码、名称、地点、负责人、预算等 |

### 在 IDEA 里打开

1. `File → Open`，选中本工程根目录的 `pom.xml`，选 **Open as Project**；
2. 等右下角 Maven 导入完成；
3. 打开 `src/main/java/com/lab/LabApplication.java`，**右键 → Run**；
4. 控制台会依次打印两个表的结构与数据、条件查询、分页、部门统计、N+1 对照、
   MyBatis-Plus 逻辑删除、以及"插入→更新→删除→回滚"的写操作演示，最后给出通过/失败统计。

连接参数在 `src/main/resources/db.properties`（已被 `.gitignore` 排除），
换机器时改这个文件，或用 `-Ddb.url=... -Ddb.password=...` 覆盖。

### 跑测试

```bash
mvn test                                  # 全量：116 个用例
mvn test -Dtest=EmpMapperTest             # 单个测试类
```

---

## 二、工程结构

```
emp-mybatis/
├── pom.xml                          单模块工程（groupId: com.lab）
├── .mvn/  mvnw  mvnw.cmd            Maven Wrapper（不用装 Maven 也能构建）
├── .gitignore  .gitattributes
├── AI代码甄别记录.md                  AI 辅助编码时的判断与修正记录
├── sql/
│   ├── 01_schema.sql                建库建表：emp(17 列) + sys_dept(8 列)
│   ├── 02_seed.sql                  种子数据：6 部门 + 16 员工
│   └── 03_verify.sql                基线核对 + 索引可用性检查
└── src/
    ├── main/java/com/lab/
    │   ├── LabApplication.java      可运行入口
    │   ├── common/                  实体 / DTO / 工具 / 异常 / 连接参数解析
    │   ├── mybatis/                 原生 MyBatis：会话工厂、Mapper、慢 SQL 插件、Service
    │   └── plus/                    MyBatis-Plus：配置工厂、BaseMapper
    ├── main/resources/
    │   ├── mybatis-config.xml       全局配置（只出现 ${db.*} 占位符）
    │   ├── log4j.properties         测试用日志（Mapper 包开 DEBUG）
    │   ├── log4j-demo.properties    演示用日志（关掉 SQL 明细）
    │   ├── db.properties.example    凭据模板（真实 db.properties 已 gitignore）
    │   └── com/lab/mybatis/mapper/  映射文件与接口同包同路径
    └── test/java/com/lab/           15 个测试类 / 116 个用例
```

---

## 三、验收要点

| 项 | 结果 |
| --- | --- |
| 构建 | `mvn -DskipTests install` 通过 |
| 测试 | 15 个测试类 / 116 个用例，Failures 0 / Errors 0 |
| 数据基线 | 物理 16 行 / 未删除 15 行 / 已逻辑删除 1 行 / 在职 12 行 / 离职 3 行 |
| N+1 对照 | 嵌套写法 7 条 SQL vs JOIN 写法 1 条，数据逐字段一致 |
| 索引 | `EXPLAIN` 实测命中 `idx_emp_status_del` 组合索引 |
| 禁用 Spring | `NoSpringDependencyTest` 运行期校验通过 |

---

## 四、安全约定

- 连接凭据一律走 `-D` 系统属性或环境变量，配置文件只允许出现 `${...}` 占位符；
- `db.properties`、`jdbc.properties`、`.env`、`target/`、`_runtime/` 均已被 `.gitignore` 排除；
- `SecurityScanTest` 把这些要求写成了检查。
