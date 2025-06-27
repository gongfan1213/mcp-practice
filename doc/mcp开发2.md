![image](https://github.com/user-attachments/assets/31f0edec-e7ee-453a-a492-177cae88e4e1)
![image](https://github.com/user-attachments/assets/31f0edec-e7ee-453a-a492-177cae88e4e1)



以下是对视频《【大模型应用开发-MCP系列】02 自定义MCP Server 并同时使用多个MCP Server全流程实操演示》的详细内容讲解，结合字幕、代码逻辑和核心知识点展开：


### **一、视频背景与目标**
#### 1. **系列定位**
   - 属于“大模型应用开发-MCP系列”，聚焦**模型上下文协议（MCP）**的实战开发，旨在通过具体案例演示如何将工具或API封装为MCP Server，并实现多Server协同调用。

#### 2. **本期目标**
   - **核心任务**：自定义一个**四则运算MCP Server**，并演示如何让MCP Client同时调用多个MCP Server（如高德地图Server和自定义Server）。
   - **前置知识**：需先掌握上期视频中“高德地图MCP Server”的使用方法（涉及地理编码、路径规划等12个接口）。


### **二、项目准备与源码获取**
#### 1. **源码结构**
   - 项目地址：视频简介中提供的GitHub/Gitee仓库（如`MCPServerTest`）。
   - 本期代码位于`02`文件夹，包含：
     - **自定义四则运算Server脚本**（Python实现）。
     - **测试脚本**：用于验证单个Server接口及多Server协同调用。

#### 2. **环境搭建**
   - **操作步骤**：
     1. 下载`02`文件夹代码，拷贝至上期项目根目录（基于上期环境扩展）。
     2. 无需重新搭建环境，直接复用上期的MCP Client配置。


### **三、自定义四则运算MCP Server开发**
#### 1. **核心代码逻辑**
   - **框架选择**：使用官方提供的`FastMCP`类（高层封装，简化开发流程）。
   - **代码结构**：
     ```python
     from mcp import FastMCP

     # 实例化FastMCP对象
     mcp = FastMCP(
         server_name="四则运算Server",
         server_description="提供加减乘除四则运算工具",
         port=8002  # 自定义端口
     )

     # 定义加法工具（通过注解声明）
     @mcp.tool(
         name="ADD",
         description="执行加法运算",
         parameters=[
             {"name": "num1", "type": "number", "description": "第一个加数"},
             {"name": "num2", "type": "number", "description": "第二个加数"}
         ]
     )
     def add(num1: float, num2: float) -> float:
         return num1 + num2

     # 同理定义减法、乘法、除法工具（SUBTRACT、MULTIPLY、DIVIDE）

     if __name__ == "__main__":
         mcp.run()  # 启动Server
     ```
   - **关键注解**：
     - `@mcp.tool`：声明工具函数，指定工具名称、描述、参数类型及格式（供大模型解析）。
     - 参数需严格定义`type`（如`number`）和`description`（影响大模型调用准确性）。

#### 2. **工具列表与接口详情**
   - 通过脚本启动后，可输出工具列表（保存至`tools.txt`）：
     ```
     工具名称：ADD
     描述：执行加法运算
     参数：num1（number，必填）、num2（number，必填）
     
     工具名称：SUBTRACT
     描述：执行减法运算
     参数：num1（number，必填）、num2（number，必填）
     ...（乘法、除法同理）
     ```
   - **大模型调用逻辑**：大模型根据用户查询（如“3+4”）解析出工具名称`ADD`和参数`3、4`，通过MCP协议调用Server接口。


### **四、单个MCP Server测试**
#### 1. **测试脚本逻辑**
   - 脚本位于`02/test_calculator_server.py`，核心步骤：
     1. **配置Server参数**：
        ```python
        server_config = {
            "name": "calculator",
            "type": "MCP",
            "url": "http://localhost:8002",  # 本地启动的四则运算Server地址
            "description": "四则运算服务"
        }
        ```
     2. **获取工具列表**：调用`mcp.get_tools()`验证工具是否正确注册。
     3. **接口测试**：
        ```python
        # 测试加法
        result = mcp.invoke_tool("ADD", {"num1": 3, "num2": 4})
        print(f"3+4={result}")  # 输出7
        ```
   - **日志与输出**：脚本运行后打印工具详情，并将结果存入文件，验证接口可用性。


### **五、多MCP Server协同调用（高德地图+自定义）**
#### 1. **MCP Client配置扩展**
   - 修改上期的Client配置，新增四则运算Server：
     ```python
     servers = [
         {
             "name": "gaode_map",
             "type": "MCP",
             "url": "http://localhost:8001",  # 高德地图Server地址
             "api_key": "你的高德API_KEY"  # 高德需认证参数
         },
         {
             "name": "calculator",
             "type": "MCP",
             "url": "http://localhost:8002"  # 自定义Server地址
         }
     ]
     ```

#### 2. **测试场景演示**
   - **场景1：调用高德地图逆地理编码**  
     **用户问题**：“经纬度30.123,120.456对应的位置是哪里？”  
     - Client解析后调用高德地图`REVERSE_GEOCODING`接口，返回地址信息。

   - **场景2：调用自定义四则运算**  
     **用户问题**：“请调用工具计算3×2”  
     - Client解析后调用`MULTIPLY`工具，返回结果6。

   - **关键观察**：  
     - 大模型会根据问题类型自动选择合适的Server（如地理问题→高德，计算问题→自定义）。  
     - 若问题简单（如“3×2”），大模型可能直接回答，需通过Prompt强制要求调用工具（如“请调用工具计算”）。


### **六、MCP协议原理与应用价值**
#### 1. **核心概念**
   - **MCP（模型上下文协议）**：  
     一种标准化工具调用协议，允许大模型通过统一接口调用外部工具/API，解耦模型与工具实现，支持跨语言、跨服务集成。

#### 2. **技术优势**
   - **标准化封装**：无论工具是本地函数（如四则运算）还是远程API（如高德地图），均通过MCP协议统一暴露接口。  
   - **可扩展性**：MCP Client可无缝接入多个Server，理论上支持无限工具扩展（如后续接入数据库、搜索等Server）。  
   - **厂商应用趋势**：当前主流厂商（如高德、百度等）正将服务封装为MCP Server，供开发者调用，降低大模型应用开发门槛。


### **七、总结与扩展**
#### 1. **本期重点**
   - 掌握通过`FastMCP`自定义Server的流程（注解定义工具、参数规范）。  
   - 理解多Server协同调用的配置方法（Client端添加Server列表）。

#### 2. **后续方向**
   - **高级功能**：后续视频将介绍MCP的`resource`（资源管理）和`prompt`（提示词优化）模块，提升工具调用效率。  
   - **传输模式**：MCP支持`STDIO`、`HTTP+SSE`、`Streamable HTTP`等模式，可根据场景选择（如实时性要求高的场景用SSE）。  
   - **实战案例**：演示更多行业场景（如数据库操作、内容搜索等）的MCP Server开发。

#### 3. **源码与资料**
   - 项目地址：[GitHub](https://github.com/NanGePlus/MCPServerTest) / [Gitee](https://gitee.com/NanGePlus/MCPServerTest)  
   - 配套文档：`02`文件夹内的`工具说明.txt`详细记录接口参数，可直接用于大模型提示词设计。


### **附：关键术语解释**
- **MCP Server**：遵循MCP协议的工具服务器，负责提供具体功能（如计算、地图查询）。  
- **MCP Client**：大模型与Server之间的桥梁，解析用户问题并路由至合适的Server。  
- **FastMCP**：MCP官方提供的Python开发框架，简化Server开发流程，底层封装网络通信与协议解析。

通过本期实践，开发者可初步掌握MCP生态的核心开发逻辑，为构建复杂大模型应用（如多工具协同的智能体）奠定基础。
