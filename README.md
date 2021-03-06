# goadmin
1. goadmin可根据组织结构一键生成具有多租户权限管理功能的脚手架，含前端和后端代码
2. goadmin可根据表结构配置一键生成基础的增删改查前端和后端代码
3. 前端代码使用Vue、ElementUI、vue-element-admin等开源项目
4. 后端代码使用gin、xormplus、casbin、gf、jwt等开源项目

# 多租户多层级权限架构
1. 例如我的系统层级结构是：平台->银行->服务商->商家，我们暂且叫这四个层级为机构类型，如下图：
![权限架构](https://images.gitee.com/uploads/images/2019/0718/093322_313ae8bc_88608.jpeg "")

2. 例如现在平台下有10个银行，每个银行有20个服务商，每个服务商有30个商户，并且要求每个银行、服务商、商户都有自己的角色、权限、用户管理

### 按上面这种业务层级机构来使用goadmin完成权限管理，使用步骤如下：
1. git clone https://github.com/wangwei123/goadmin
2. cd goadmin/configs
3. 修改mysql.toml，将mysql配置信息改为你的数据库信息，数据库名称可随意
4. 修改casbin.toml，将mysql配置信息改为你的数据库信息，且数据库名称必须为casbin
5. 修改org_type.json, 这个文件中存的是机构类型，有层级关系，配置如下:  
```
[
  { "id":1, "parent_id":0, "name":"平台",   "code":"platform" },
  { "id":2, "parent_id":1, "name":"银行",   "code":"bank" },
  { "id":3, "parent_id":2, "name":"服务商", "code":"service_provider" },
  { "id":4, "parent_id":3, "name":"商户",   "code":"shop" }
]
```
6. 修改org.json, 这个文件中存的是机构的数据库字段定义，配置如下:  
```
// "code": "string,50,银行编码,1"
// code为数据库字段名称，string是golang中的数据类型,50是字段长度,银行编码是字段描述,1是字段顺序
{
  "bank": {
    "code":           "string,50,银行编码,1",
    "name":           "string,80,银行名称,2",
    "contact_name":   "string,40,联系人,3",
    "service_phone":  "string,20,联系电话,4",
    "org_type_id":    "int64, 20,所属机构类型ID,5",
    "org_type_name":  "string,40,所属机构类型名称,6",
    "account":        "string,40,管理员账号,7"
  },
  "service_provider": {
    "name":           "string,80,服务商名称,1",
    "contact_name":   "string,40,联系人,2",
    "service_phone":  "string,20,联系电话,3",
    "address":        "string,80,联系地址,4",
    "org_type_id":    "int64, 20,所属机构类型ID,5",
    "org_type_name":  "string,40,所属机构类型名称,6",
    "account":        "string,40,管理员账号,7"
  },
  "shop": {
    "name":           "string,80,商家名称,1",
    "contact_name":   "string,40,联系人,2",
    "service_phone":  "string,20,服务电话,3",
    "address":        "string,80,商家地址,4",
    "org_type_id":    "int64, 20,所属机构类型ID,5",
    "org_type_name":  "string,40,所属机构类型名称,6",
    "account":        "string,40,管理员账号,7"
  },
  "desc": {
    "bank": "银行",
    "service_provider": "服务商",
    "shop": "商家"
  }
}
```
7. 创建hello项目
```
cd goadmin/cmd

go build goadmin.go

// MacOS/Linux
./goadmin -newProject hello

// Windows  windows系统下建议使用git bash命令行工具,下载地址：https://git-scm.com/download/win
./goadmin.exe -newProject hello
```
8. 查看并启动生成的代码
```
// 启动生成的go程序
cd goadmin/output

// 进入项目根目录
cd hello-go

// 下载依赖包
go mod tidy

// 运行
cd cmd && go run main.go


// 启动生成的前端Vue程序
cd goadmin/output

cd hello-admin

npm install && npm run dev
```
9. 初始化超级管理员账号密码为：super_admin/111111



### 如何自动生成一个基础的增删改查前后端代码：
```
1. 首先在configs/new_gen_module.json中配置数据库表结构，例如：
{
  "student": {
    "code":           "string,50,银行编码,1",
    "name":           "string,80,银行名称,2",
    "contact_name":   "string,40,联系人,3",
    "service_phone":  "string,20,联系电话,4",
    "org_type_id":    "int64, 20,所属机构类型ID,5",
    "org_type_name":  "string,40,所属机构类型名称,6",
    "account":        "string,40,管理员账号,7"
  },
  "teacher": {
    "name":           "string,80,服务商名称,1",
    "contact_name":   "string,40,联系人,2",
    "service_phone":  "string,20,联系电话,3",
    "address":        "string,80,联系地址,4",
    "org_type_id":    "int64, 20,所属机构类型ID,5",
    "org_type_name":  "string,40,所属机构类型名称,6",
    "account":        "string,40,管理员账号,7"
  },
  "class": {
    "name":           "string,80,商家名称,1",
    "contact_name":   "string,40,联系人,2",
    "service_phone":  "string,20,服务电话,3",
    "address":        "string,80,商家地址,4",
    "org_type_id":    "int64, 20,所属机构类型ID,5",
    "org_type_name":  "string,40,所属机构类型名称,6",
    "account":        "string,40,管理员账号,7"
  },
  "desc": {
    "student": "学生",
    "teacher": "老师",
    "class": "班级"
  }
}

2. 然后只需要执行如下命令, 输入项目名称projectName即可，这里项目名称是hello
// MacOS
./goadmin -newModule -projectName hello

// Windows
./goadmin.exe -newModule -projectName hello

3. 在output目录下会生成gencodes文件夹，包含了后端代码gocode目录，前端代码vuecode目录

```

有疑问可以加我微信：  
![微信二维码](https://images.gitee.com/uploads/images/2019/0718/094023_81e9896e_88608.png "微信二维码")



