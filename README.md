根据您提供的文档内容和要求，我来整理完整的 HBase Shell 操作命令：

## 1. 创建表

### 1.1 创建学生表（Student）
```bash
create 'Student', 'S_No', 'S_Name', 'S_Sex', 'S_Age'
```

### 1.2 创建课程表（Course）
```bash
create 'Course', 'C_No', 'C_Name', 'C_Credit'
```

### 1.3 创建选课表（SC）
```bash
create 'SC', 'SC_Sno', 'SC_Cno', 'SC_Score'
```

## 2. 插入数据

### 2.1 向学生表插入数据
```bash
put 'Student', '2015001', 'S_Name', 'Zhangsan'
put 'Student', '2015001', 'S_Sex', 'male'
put 'Student', '2015001', 'S_Age', '23'

put 'Student', '2015002', 'S_Name', 'Mary'
put 'Student', '2015002', 'S_Sex', 'female'
put 'Student', '2015002', 'S_Age', '22'

put 'Student', '2015003', 'S_Name', 'Lisi'
put 'Student', '2015003', 'S_Sex', 'male'
put 'Student', '2015003', 'S_Age', '24'
```

### 2.2 向课程表插入数据
```bash
put 'Course', '123001', 'C_Name', 'Math'
put 'Course', '123001', 'C_Credit', '2.0'

put 'Course', '123002', 'C_Name', 'Computer Science'
put 'Course', '123002', 'C_Credit', '5.0'

put 'Course', '123003', 'C_Name', 'English'
put 'Course', '123003', 'C_Credit', '3.0'
```

### 2.3 向选课表插入数据
```bash
put 'SC', '2015001_123001', 'SC_Score', '86'
put 'SC', '2015001_123003', 'SC_Score', '69'
put 'SC', '2015002_123002', 'SC_Score', '77'
put 'SC', '2015002_123003', 'SC_Score', '99'
put 'SC', '2015003_123001', 'SC_Score', '98'
put 'SC', '2015003_123002', 'SC_Score', '95'
```

## 3. 查询操作

### 3.1 查看单个学生信息
```bash
get 'Student', '2015001'
```

### 3.2 查看所有学生信息
```bash
scan 'Student'
```

### 3.3 查看特定课程信息
```bash
get 'Course', '123001'
```

### 3.4 查看所有课程
```bash
scan 'Course'
```

### 3.5 查看选课记录
```bash
scan 'SC'
```

## 4. 删除操作

### 4.1 删除单个数据
```bash
# 删除学生2015001的年龄信息
delete 'Student', '2015001', 'S_Age'
```

### 4.2 删除整行数据
```bash
# 删除学生2015003的所有信息
deleteall 'Student', '2015003'
```

## 5. 表管理操作

### 5.1 查看表结构
```bash
describe 'Student'
```

### 5.2 禁用表（删除前必需）
```bash
disable 'Student'
```

### 5.3 删除表
```bash
drop 'Student'
```

### 5.4 清空表（保留表结构）
```bash
truncate 'Student'
```

## 6. 高级查询

### 6.1 查询特定学生的选课成绩
```bash
# 查询学号以2015001开头的选课记录
scan 'SC', {STARTROW => '2015001', ENDROW => '2015002'}
```

### 6.2 启用表
```bash
enable 'Student'
```

### 6.3 检查表状态
```bash
is_enabled 'Student'
is_disabled 'Student'
```

这些命令涵盖了 HBase Shell 的基本操作，包括表的创建、数据插入、查询、删除和管理等功能。您可以根据实际需要选择相应的命令来操作 HBase 数据库。
