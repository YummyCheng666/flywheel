# 环境准备
## 运行环境
### JDK 版本与安装
- JDK 1.8 以上版本
- Mac JDK 安装,请参考：https://www.codenong.com/24342886/
### Maven 
- Apache Maven 3.5.3
- Maven 需要替换setting.xml 文件，联系服务端开发/测试 要一个
### 代码编译开发工具
- idea
- 一般需要付费版，请google 破解方式，或者找服务端开发/测试要一个破解版

# 接口测试框架架构
### 使用技术
- Springboot
- TestNg
- DataProvider
### 代码结构
    |-- glass-saas-api-test -------------------- 项目根目录      
        |-- .gitignore      -------------------- 配置git 忽略提交的文件，以防将编译自动生成的文件提交到git 上  
        |-- README.md
        |-- SAAS平台API文档0.9.1.pdf  ----------- 接口文档，按照文档写用例场景
        |-- pom.xml     ------------------------ 项目运行需要的jar 全部通过maven 引用构建，mvn clean、mvn package 等基本命令需要掌握
        |-- 环境与架构.md     ------------------- 看项目先看md
        |-- src                  
            |-- main
                |-- java      ------------------ 不解释，java 项目创建必有src/main/java
                |   |-- com
                |       |-- rokid   ------------ com.rokid 是基础包名
                |           |-- Application.java    --------------- Spring boot 启动类，没事儿不要改
                |           |-- bean    ----------------------- 测试Excel 文件读取转换基类，和Excel 中定义数据参数一一匹配，不经过允许，不准擅自修改
                |           |   |-- ApiDataBean.java
                |           |   |-- BaseBean.java
                |           |-- cases ------------------------- 测试用例
                |           |   |-- BaseTest.java ------------- 测试用例基类，所有测试用例的要继承BaseTest
                |           |   |-- grpc
                |           |   |-- http  --------------------- http接口测试用例，按照场景新增包名，新增用例
                |           |       |-- auth
                |           |           |-- AuthTest.java ---- 示例case，远程协作手机账户登陆-修改密码-退出场景
                |           |-- config   --------------------- 预加载的固定参数，和yml 文件配合使用
                |           |   |-- CommonConfig.java -------- 配置接口地址，设备sn，登陆用户名，密码，appkey 等常用不变的的配置相，自动装载yml
                |           |-- functions  ------------------- 基础方法，配合util/FunctionUtil.java 将A=B 转换为 key，value 格式
                |           |   |-- Function.java
                |           |-- listeners  ------------------- 基础方法，优化TestNg 报告展示方式
                |           |   |-- ExtentTestNGIReporterListener.java
                |           |-- provider  ------------------- 基础方法，读取Excel 数据作为数据驱动
                |           |   |-- DataFileParameters.java
                |           |   |-- ExcelDataProvider.java
                |           |-- util     ------------------- 基础方法类库，方法定义为static
                |               |-- AuthorizationUtil.java  --------- 生成接口访问header 中的Authorization
                |               |-- ClassFinder.java
                |               |-- DateUtil.java   ---------------- 日期处理公共类，对日期格式进行各种转换
                |               |-- DecodeUtil.java  --------------- 转码公共类
                |               |-- EncryptionAndDecryptUtil.java -- 加解密公共类
                |               |-- ExcelUtil.java  ---------------- Excel 读取
                |               |-- FileUtil.java   ---------------- 文件路径获取
                |               |-- FunctionUtil.java -------------- A=B 转换为key、value
                |               |-- HttpUtils.java ----------------- http 请求提交
                |               |-- RandomUtil.java ---------------- 随机字符串生成
                |-- resources
                    |-- log4j.properties -------------------------- log 日志打印配置
                    |-- casedata  --------------------------------- 测试数据文件夹
                    |   |-- auth.xlsx ----------------------------- 和AuthTest 配合，测试数据
                    |   |-- logout.xlsx
                    |   |-- updatePwdStrong.xlsx
                    |-- config  ----------------------------------- 配置文件
                    |   |-- application-dev.yml ------------------- 测试环境配置文件，配置测试环境访问路径，sn，账户，设备
                    |   |-- application-pro.yml ------------------- 线上环境配置文件
                    |   |-- application.yml ----------------------- spring boot 主启动文件，里面配置加载dev or pro 的yml
                    |-- testng ------------------------------------ testng 配置文件
                        |-- companyAccountTestng.xml
                        |-- devicetestng.xml
                        |-- testng.xml --------------------------- 配置要运行的test 类
### 数据驱动Excel 表格结构
|step|run|desc|url|method|header|param|verify|save|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|步骤|是否运行|描述|接口访问地址|http 请求类型|自定义请求头|参数|要验证的数据|保存结果下一个用例使用|
|1|Y|PC账户登陆|/api/auth/oauth/token|post| |{</br>"appKey":"${pcAppKey}",</br>"grantType":"companyUser",</br> "companyIndex":"${companyIndex}",</br> "deviceUserName":"${deviceUserName}",</br> "password":"${password}"</br> }|$.code=1|accessToken=$.data.accessToken;</br> refreshToken=$.data.refreshToken|
|2|N|设备登陆|/api/auth/oauth/token|post| |{</br>"appKey":"${deviceAppKey}",</br> "grantType":"device",</br>  "deviceTypeId":"${deviceTypeId}",</br> "deviceId":"${deviceId}"</br> }|$.code=1|accessToken=$.data.accessToken;</br> refreshToken=$.data.refreshToken|
|3|Y|修改密码|/api/upms/v1/deviceUser/changePwd|put|authorization=Bearer ${accessToken}|{</br>"oldPwd":"Rokid@1234",</br>"newPwd":"Rokid@12345"</br>}|$.code=1| |
|4|Y|修改密码|/api/upms/v1/deviceUser/changePwd|put|authorization=Bearer ${accessToken}|{</br>"oldPwd":"Rokid@12345",</br>"newPwd":"Rokid@1234"</br>}|$.code=1| |
|5|Y|退出登录|/api/auth/token/logout|delete|authorization=Bearer ${accessToken}| |$.code=1| |

# 接口测试
## 基本准备
- 查看接口文档，在脑子对接口进行组装
- 接口+接口 组成场景，这一步要想清楚
## 开始写代码
- 1、在 resources/casedata中新建一个excel，以英文命名
- 2、按照接口定义，在excel 中开始写用例
- 3、写完后，在src/main/java/rokid/cases/http 中新增一个package
- 4、在新增的package 中new 一个java Class，继承BaseTest
     ```java
     public class AuthTest extends BaseTest {
     }
     ```
- 5、写一个方法，装载Excel 并调用BaseTest 中的runTest
     ```java
     public class AuthTest extends BaseTest {
         @Test(dataProvider="excelDataProvider",dataProviderClass= ExcelDataProvider.class)
         @DataFileParameters(fileName = "auth.xlsx", path = "src/main/resources/casedata", sheetName = "Sheet1")
         public void  companyUserAuthTest(ApiDataBean apiDataBean) throws Exception {
              runTest(apiDataBean);
         }
     }
     ```
- 6、右键run test，检查case 结果
- 7、Cases 运行完成后，配置testNg.xml 运行即可
- 8、如何配置testNg.xml ,请参考：https://www.yiibai.com/testng/expected-exception-test.html
    
