# Code-auto

通过命令行自动根据自定义模板生成代码文件

## Install

    *./codeAuto.js*
    npm install  code-auto

    var codeAuto=require("code-auto");
    const config = {
            dialect: "mysql",   数据库类型（mysql，Postgres，Sqlite3，MSSQL）
            port: '3306', 端口（仅mysql和Postgres）
            host:  "127.0.0.1",  数据库服务器地址
            username:  "root",   数据库账号
            pwd: "password",     数据库密码
            storage: "database", 数据库名
            Templates: './codetpl',  模板文件夹路径
            codetype: "js"  代码类型（决定columns变量中CodeType） （ c# || java || js ）
        }
    codeAuto.run(config,function(err){
        if(err){
        throw new Error(err);
        }
        console.log("完成")
    })

    $node node codeAuto.js

## 依赖

在使用Code-auto 你需要在全局安装正确你使用的数据库

Example for MySQL/MariaDB

`npm install -g mysql`

Example for Postgres

`npm install -g pg pg-hstore`

Example for Sqlite3

`npm install -g sqlite`

Example for MSSQL

`npm install -g mssql`

## 模板内置变量

        tableName：表名

        columns：当前表的所有列
        columns item:
                name 列名
                type 数据类型
                CodeType 代码内使用的数据类型（根据配置代码类型生成（ c# || java || js ））
                primaryKey (false || true) 是否主键
                allowNull (false || true) 是否可以为NULL
                defaultValue 默认值


## 模板定义


模板必须包含在template内，同时在template上定义output属性做为生成输出路径


## 例


    <template output='./model'>
        /* jshint indent: 2 */
            module.exports = app => {
            const sequelize = app.Sequelize;
            const entity = {
                {{#columns.forEach(function(column,index){ }}
                    {{column.name}}: {
                        type: {{column.CodeType}},
                        allowNull: {{column.allowNull}},
                        {{column.primaryKey?"primaryKey:"+column.primaryKey+",":""}}
                        {{column.autoIncrement?"autoIncrement:"+column.autoIncrement:""}}
                    }{{index==(columns.length-1)?'':','}}
                {{# }); }}
            }
            const nav_cat = app.model.define('{{tableName}}', entity, {
                tableName: '{{tableName}}'
            });
            {{tableName}}.getModel=model=>{
                                        if(typeof model!=="object"){
                                            throw new  Error("请求参数错误");
                                            return false;
                                        }
                                        let newobj={};
                                        for (var key in model) {
                                            if (entity.hasOwnProperty(key)) {
                                                newobj[key]=model[key];
                                            }
                                        }
                                    return newobj;
                                    }
            return {{tableName}};
            };
    </template>