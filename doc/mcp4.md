![image](https://github.com/user-attachments/assets/7d492530-b5a7-4dda-9214-4c2f9838ec2b)



该视频是“大模型应用开发-MCP系列”的第4期，主要围绕使用STDIO传输模式及底层MCP SDK实现MySQL MCP Server展开，涵盖从环境搭建到功能实现的全流程操作与原理讲解。以下是详细总结归纳：


### **一、核心目标与内容规划**
- **目标**：  
  使用MCP官方底层SDK（非FastMCP框架）和**STDIO传输模式**，开发一个支持MySQL数据库访问的MCP Server，实现数据表资源的**增删改查、联表查询及SQL语句执行**功能。
- **系列规划**：  
  本期聚焦STDIO模式，后续两期将分别演示**HTTP+SSE模式**和**Streamable HTTP模式**，使用相同MySQL案例，代码已整理完成。


### **二、前期准备工作**
1. **环境与依赖**  
   - **开发环境**：需参考前序视频搭建MCP开发环境（如项目初始化、大模型服务接口配置）。  
   - **源码获取**：从视频简介下载对应源码（分3个文件夹对应3种传输模式），拷贝到已创建的`mcpservertest`项目根目录。  
   - **依赖升级**：  
     - MCP SDK版本升级至**1.8.0**（此前为1.7版本），通过命令行执行升级。  
     - 安装MySQL相关依赖包（如`mysql-connector-python`）。  

2. **Docker与MySQL服务启动**  
   - 下载安装Docker，启动Docker服务。  
   - 通过命令行进入包含Docker配置文件（`docker-compose.yml`）的目录，运行指令拉取MySQL镜像并启动服务。  
   - 默认配置：用户名`nange`，密码`123456`，数据库名为`test_db`（在配置文件中定义）。  

3. **测试数据导入**  
   - 使用数据库客户端（如Navicat）连接本地MySQL（端口默认3306），导入视频提供的两个SQL文件：  
     - **学生信息表（student_info）**：字段包含学号、姓名、年龄。  
     - **学生成绩表（student_score）**：字段包含学号、成绩，与学生表通过学号关联。  


### **三、MCP Server功能与实现逻辑**
#### **1. 核心功能**
- **资源（Resources）**：对外暴露数据库中的数据表，支持通过URI访问表结构和数据。  
  - 示例URI格式：`mcp://mysql/student_info/data`（表示学生信息表的数据）。  
- **工具（Tools）**：提供SQL语句执行功能，支持查询、增删改操作及联表查询。  
  - 工具名称：`execute_sql`，参数为`query`（SQL语句）。  

#### **2. 技术实现**
- **传输模式**：STDIO（标准输入输出），通过子进程与主进程的流通信实现数据交换。  
- **底层SDK使用**：  
  - 通过`mcp.run`方法初始化MCP Server，传入输入流（`sys.stdin`）和输出流（`sys.stdout`）。  
  - 定义资源列表接口（`list_resources`）：查询数据库所有表名，封装为MCP标准资源格式（包含URI、名称、描述等）。  
  - 定义资源读取接口（`get_resource`）：解析URI获取表名，执行SQL查询并返回数据（默认返回前5条）。  
  - 定义工具接口（`list_tools`和`execute_tool`）：  
    - `list_tools`返回工具元数据（名称、描述、参数类型）。  
    - `execute_tool`根据传入的SQL语句类型（查询、增删改）执行对应操作，返回结果或影响行数。  

- **代码结构**：  
  - 入口脚本（`main.py`）：加载数据库配置，初始化MCP Server，注册资源和工具处理函数。  
  - 测试脚本（`test_client.py`）：使用STDIO模式的MCP Client连接Server，测试资源和工具接口。  


### **四、操作演示与测试**
1. **Docker与数据库初始化**  
   - 启动Docker后，通过命令行执行`docker-compose up`拉取并运行MySQL容器。  
   - 使用客户端验证连接成功，导入测试表后刷新数据库，确认两张表存在且数据正确。  

2. **接口测试**  
   - 运行测试脚本`test_client.py`，输出MCP Server提供的资源和工具列表：  
     - 资源列表：显示`student_info`和`student_score`两张表。  
     - 工具列表：显示`execute_sql`工具，参数为`query`。  
   - 测试资源访问：通过URI查询表数据，如`mcp://mysql/student_info/data`返回学生信息。  
   - 测试工具调用：执行SQL语句，如`SELECT * FROM student_score WHERE score > 90`，返回符合条件的记录。  

3. **大模型集成演示**  
   - 运行大模型交互脚本，通过自然语言提问触发工具调用：  
     - 提问“查询成绩最高的学生”，大模型生成SQL`SELECT * FROM student_score ORDER BY score DESC LIMIT 1`并执行。  
     - 提问“将张三的名字改为钱八”，执行UPDATE语句并验证数据库更新结果。  
   - 多轮交互：大模型根据问题自动拼接SQL，调用工具后整理结果返回。  


### **五、关键知识点与对比**
- **STDIO模式特点**：  
  - 通过子进程（Server）与主进程（Client）的标准流通信，适合本地调试和简单场景。  
  - 代码需显式处理输入输出流，灵活性高但复杂度高于FastMCP框架。  
- **与FastMCP的区别**：  
  - FastMCP封装了底层细节，适合快速开发；本期使用底层SDK，可自定义传输逻辑和协议细节。  
- **资源与工具的标准化**：  
  - 需遵循MCP协议定义的JSON RPC 2.0格式，确保与大模型或其他客户端兼容。  


### **六、延伸与后续内容**
- **代码复用性**：后续视频将基于本期代码，仅修改传输模式相关逻辑（如HTTP客户端初始化），其余功能代码保持不变。  
- **学习建议**：  
  - 提前掌握MCP基础概念（如传输模式、资源/工具定义），参考前序视频《MCP三种传输模式详解》。  
  - 结合源码理解STDIO模式的进程间通信原理，为后续HTTP模式学习打下基础。  

该视频通过实操演示，系统讲解了如何使用底层SDK开发数据库型MCP Server，适合希望深入理解MCP协议底层机制的开发者。
