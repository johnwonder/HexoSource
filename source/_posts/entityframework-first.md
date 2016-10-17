title: entityframework_随想
date: 2016-08-26 13:51:05
tags:
---

## EntityFramework之DbContext
### 1. IdentityDbContext 貌似在AutomaticMigrationsEnabled=false时仍然可以生成数据库。DbContext貌似就不可以。

### 2. AutomaticMigration为true时删除数据库某个表的字段后再新增实体的属性时，有可能会报data loss之类的错误，可能跟_MigrationHistory表有关。

### 3. 把_Migration表清空后，如果对象新增属性，会报数据库中已存在名为 '' 的对象的错误 。

一开始如果没有用AutomaticMigration生成数据库，之后新增对象属性，如果配置了Database.SetInitializer 那么会报已存在""的对象。
	