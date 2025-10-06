# ECS 服务器后端搭建教程

本教程基于 **华为云 ECS 服务器**，主要介绍从服务器购买、文件传输，到 Python 脚本运行、Nginx 配置、网页部署以及 SSL 域名配置的完整流程。  
适用于搭建小型后端服务（如微信小程序的后端接口）。

---

## 一、购买和登录 ECS 服务器

ECS 服务器相当于一台远程电脑，可以长期运行脚本和程序。  
虽然性能通常不如本地电脑，但优势在于 **7×24 小时在线**。

- 登录 [华为云官网](https://www.huaweicloud.com)，搜索 **ECS 服务器** 业务并购买。推荐配置：  
  - 地区：华北-北京四  
  - 规格：通用计算增强型 | 2 vCPUs | 4 GiB  
  - 系统：Windows Server 2019 Standard 64bit  
  - 公网访问：按流量计费（带宽仅作限速，不作为收费依据）

- 进入 **ECS 控制台 → 重置密码**，设置新密码。  
- 使用 **远程桌面 (MSTSC)** 登录服务器。  

👉 详细步骤可参考华为云文档：[快速购买和使用 Windows ECS](https://support.huaweicloud.com/qs-ecs/ecs_01_0102.html#ecs_01_0102__section1941981618913)

---

## 二、文件上传与下载

由于云厂商一般限制浏览器直接下载软件，推荐使用 **OBS（对象存储服务）** 进行文件传输。

1. 购买 **OBS 标准存储多 AZ 包**（存储规格以内免费，超出按需计费）。  
2. 使用 **MSTSC** 将 **OBS Browser+** 安装包上传到服务器。  
   参考文档：[通过 MSTSC 上传文件](https://support.huaweicloud.com/ecs_faq/ecs_faq_0800.html)  
3. 登录 **OBS Browser+**（永久 AK 方式）：  
   - 账号名：自定义，仅区分本地不同账号  
   - 服务提供商：其他对象存储服务  
   - 服务器地址：华北-北京四 → `obs.cn-north4.myhuaweicloud.com`详见  [OBS服务器区域地址查询](https://console.huaweicloud.com/apiexplorer/#/endpoint/OBS)
   - Access Key ID / Secret Access Key：  
     进入「右上角用户名 → 我的凭证 → 访问密钥」获取（下载后仅能保存一次，建议添加到系统变量）  详见[AK/SK概念介绍](https://support.huaweicloud.com/productdesc-obs/obs_03_0208.html)

4. 使用 OBS Browser+ 拖拽上传或下载，实现本地与服务器间的文件传输。  

---

## 三、Python 脚本运行

在运行 Python 脚本前，需要安装 **Nginx**，用于管理端口和反向代理。  
Nginx 相当于 **服务器内部端口与外部访问的中转站**。

### 1. 获取微信小程序用户信息
- 使用 `wx.getUserProfile` 接口采集用户信息。  详见[开放接口/用户信息](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/user-info/wx.getUserProfile.html)
- 解密程序示例下载：[AES Sample](https://res.wx.qq.com/wxdoc/dist/assets/media/aes-sample.eae1f364.zip)

### 2. 接收小程序请求并存储
- 编写 **Flask 脚本**：接收 POST 请求 → 数据处理 → 保存到本地 → 上传到 OBS → 删除副本。  
- 安装依赖：
  ```bash
  pip install esdk-obs-python --trusted-host pypi.org
  ```
  详见[Python SDK接口概览](https://support.huaweicloud.com/sdk-python-devg-obs/obs_22_0100.html)
### 3.端口冲突处理
Flask 默认运行在 **5000/443 端口**，若与 Nginx 冲突，可修改 Nginx 配置映射端口。

查看端口占用：

`netstat -ano | findstr :端口号` 

结束进程：

`taskkill /PID 1234 /F`
### 4. 配置安全组

进入 **ECS 控制台 → 安全组 → 配置规则**，开放所需端口（如 80、443、5000）。

----------

## 四、网页端部署

参考以下教程进行 Nginx 部署与反向代理配置：

-   [教程一](https://blog.csdn.net/momohhhhh/article/details/126319350)
    
-   [教程二](https://blog.csdn.net/s_naughty/article/details/144981486)
    

----------

## 五、SSL 证书与域名

-   **SSL 证书**：用于启用 HTTPS，小程序必须通过 HTTPS 传输。
    
-   **域名要求**：小程序后台需备案绑定的域名。
    

### 获取方式：

1.  SSL 证书可通过 **阿里云、腾讯云** 等购买，或申请免费证书。
    
2.  域名购买：华为云已停止域名服务，需在其他云服务商购买。
    
3.  华为云 SSL 证书绑定流程可参考：快速申请和使用 OV 型证书
    

----------

## 注意事项

-   确保 **Flask 脚本运行的端口与 Nginx 配置对应**。
    
-   **安全组必须开放端口**，否则外部无法访问。
    
-   **AK/SK 文件请妥善保存**，避免泄露。
    

----------

## 总结

本教程主要涵盖以下步骤：

1.  购买并登录 ECS 服务器
    
2.  使用 OBS 进行文件传输
    
3.  部署 Python 脚本并配置 Nginx
    
4.  网页端部署与端口管理
    
5.  SSL 证书与域名绑定
