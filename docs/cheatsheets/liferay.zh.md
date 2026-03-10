# Liferay

> Liferay Portal 是一款开源的企业级门户平台，用于构建 Web 应用程序和数字体验。它提供了内容管理、用户身份验证、协作工具和可自定义的仪表板等功能。 —— [liferay/liferay-portal](https://github.com/liferay/liferay-portal)

## 目录 (Summary)

* [Portlets](#portlets)
* [登录页面 (Login Page)](#%e7%99%bb%e5%bd%95%e9%a1%b5%e9%9d%a2-login-page)
* [注册页面 (Register Page)](#%e6%b3%a8%e5%86%8c%e9%a1%b5%e9%9d%a2-register-page)
* [用户个人主页 (User Profile)](#%e7%94%a8%e6%88%b7%e4%b8%aa%e4%ba%ba%e4%b8%bb%e9%a1%b5-user-profile)
* [用户配置 (User Configuration)](#%e7%94%a8%e6%88%b7%e9%85%8d%e7%bd%ae-user-configuration)
* [控制面板 (Control Panel)](#%e6%8e%a7%e5%88%b6%e9%9d%a2%e6%9d%bf-control-panel)
* [API](#api)
* [相关漏洞 (Vulnerabilities)](#%e7%9b%b8%e5%85%b3%e6%bc%8f%e6%b4%9e-vulnerabilities)
    * [开放重定向 (Open Redirect)](#%e5%bc%80%e6%94%be%e9%87%8d%e5%ae%9a%e5%90%91-open-redirect)
    * [管理员控制面板代码执行](#%e7%ae%a1%e7%90%86%e5%91%98%e6%8e%a7%e5%88%b6%e9%9d%a2%e6%9d%bf%e4%bb%a3%e7%a0%81%e6%89%a7%e8%a1%8c)
    * [通过 I18nServlet 导致的资源泄露](#%e9%80%9a%e8%bf%87-i18nservlet-%e5%af%bc%e8%87%b4%e7%9a%84%e8%b5%84%e6%ba%90%e6%b3%84%e9%9c%b2)
    * [通过 JSON Web 服务导致的远程代码执行 (RCE)](#%e9%80%9a%e8%bf%87-json-web-%e6%9c%8d%e5%8a%a1%e5%af%bc%e8%87%b4%e7%9a%84%e8%bf%9c%e7%a8%8b%e4%bb%a3%e7%a0%81%e6%89%a7%e8%a1%8c-rce)
* [参考资料 (References)](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99-references)

## Portlets

```ps1
/?p_p_id=<portlet_ID>&p_p_lifecycle=0&p_p_state=<window_state>&p_p_mode=<mode>
```

* **portlet_ID**：要执行的 Portlet ID。可以是数字 ID（每个 Portlet 的递增数字），也可以是字符串形式的 [完全限定 Portlet ID (Fully-Qualified Portlet IDs)](https://help.liferay.com/hc/en-us/articles/360018511712-Fully-Qualified-Portlet-IDs)。

* **window_state**：Portlet 在页面上占据的空间量。取值包括：normal（正常）、maximized（最大化）、minimized（最小化）。

* **mode**：Portlet 的当前功能。取值包括：view（查看）、edit（编辑）、help（帮助）。

| 名称 | Portlet ID |
| ------------------- | ---------- |
| 资源发布器 (Asset Publisher) | com_liferay_asset_publisher_web_portlet_AssetPublisherPortlet |
| 文档与媒体 (Documents and Media) | com_liferay_document_library_web_portlet_DLPortlet |
| 导航菜单 (Navigation Menu) | com_liferay_site_navigation_menu_web_portlet_SiteNavigationMenuPortlet |
| 站点地图 (Site Map) | com_liferay_site_navigation_site_map_web_portlet_SiteNavigationSiteMapPortlet |
| Web 内容展示 (Web Content Display) | com_liferay_journal_content_web_portlet_JournalContentPortlet |
| 搜索栏 (Search Bar) | com_liferay_portal_search_web_search_bar_portlet_SearchBarPortlet |
| 搜索 (Search) | com_liferay_portal_search_web_portlet_SearchPortlet |

## 登录页面 (Login Page)

```ps1
/login
/c/portal/login
/?p_p_id=58&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view
/?p_p_id=58&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&saveLastPath=false&_58_struts_action=%2Flogin%2Flogin
/?p_p_id=com_liferay_login_web_portlet_LoginPortlet&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view
/?p_p_id=com_liferay_login_web_portlet_LoginPortlet&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&saveLastPath=false&_58_struts_action=%2Flogin%2Flogin
```

## 注册页面 (Register Page)

```ps1
/?p_p_id=58&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&_com_liferay_login_web_portlet_LoginPortlet_mvcRenderCommandName=%2Flogin%2Fcreate_account
/?p_p_id=58&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&saveLastPath=false&_58_struts_action=%2Flogin%2Flogin&_com_liferay_login_web_portlet_LoginPortlet_mvcRenderCommandName=%2Flogin%2Fcreate_account
/?p_p_id=com_liferay_login_web_portlet_LoginPortlet&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&_com_liferay_login_web_portlet_LoginPortlet_mvcRenderCommandName=%2Flogin%2Fcreate_account
/?p_p_id=com_liferay_login_web_portlet_LoginPortlet&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&saveLastPath=false&_58_struts_action=%2Flogin%2Flogin&_com_liferay_login_web_portlet_LoginPortlet_mvcRenderCommandName=%2Flogin%2Fcreate_account
```

## 用户个人主页 (User Profile)

```ps1
/web/<用户名>
/web/<用户名>/home
/user/<其他用户>/control_panel/manage
/user/<其他用户>/~/control_panel/manage
/web/guest
/web/guest/home
```

## 用户配置 (User Configuration)

```ps1
/user/<用户名>
/user/<用户名>/manage
/user/<用户名>/manage?p_p_id=com_liferay_my_account_web_portlet_MyAccountPortlet&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view
/group/control_panel/manage?p_p_id=com_liferay_my_account_web_portlet_MyAccountPo
```

## 控制面板 (Control Panel)

经过身份验证的用户可以访问的端点：

```ps1
/group/control_panel/manage
/group/guest/control_panel/manage
/group/guest/~/control_panel/manage
/group/<用户名>/control_panel/manage
/group/<用户名>/~/control_panel/manage
/user/<用户名>/control_panel/manage
/user/<用户名>/~/control_panel/manage
```

## API

* [nuclei-templates/http/misconfiguration/liferay/liferay-axis.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/liferay/liferay-axis.yaml)
* [nuclei-templates/http/misconfiguration/liferay/liferay-jsonws.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/liferay/liferay-jsonws.yaml)
* [nuclei-templates/http/misconfiguration/liferay/liferay-api.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/liferay/liferay-api.yaml)

| 名称 | 路径 |
| ----------------- | ------------- |
| JSON Web 服务 | `/api/jsonws` |
| SOAP | `/api/axis` |
| GraphQL | `/o/graphql` |
| JSON 和 GraphQL | `/o/api` |

## 相关漏洞 (Vulnerabilities)

* [liferay.dev/known-vulnerabilities (已知漏洞)](https://liferay.dev/portal/security/known-vulnerabilities)
* [ilmila/J2EEScan](https://github.com/ilmila/J2EEScan/blob/master/src/main/java/burp/j2ee/issues/impl/LiferayAPI.java)

### 开放重定向 (Open Redirect)

```ps1
/html/common/referer_jsp.jsp?referer=<url>
/html/common/referer_js.jsp?referer=<url>
/html/common/forward_jsp.jsp?FORWARD_URL=<url>
/html/common/forward_js.jsp?FORWARD_URL=<url>
```

### 管理员控制面板代码执行

Gogo shell 模式，可读取文件：

```ps1
/group/control_panel/manage?p_p_id=com_liferay_gogo_shell_web_internal_portlet_GogoShellPortlet&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&_com_liferay_gogo_shell_web_internal_portlet_GogoShellPortlet_javax.portlet.action=executeCommand
```

Groovy 解释器：

```ps1
/group/control_panel/manage?p_p_id=com_liferay_server_admin_web_portlet_ServerAdminPortlet&p_p_lifecycle=0&p_p_state=maximized&p_p_mode=view&_com_liferay_server_admin_web_portlet_ServerAdminPortlet_mvcRenderCommandName=%2Fserver_admin%2Fview&_com_liferay_server_admin_web_portlet_ServerAdminPortlet_tabs1=script
```

### 通过 I18nServlet 导致的资源泄露

Liferay 在 I18n Servlet 中存在本地文件包含漏洞，因为它通过向 `/[language]/[resource];.js`（`.jsp` 同样适用）发送 HTTP 请求来泄露信息。[nuclei-templates/http/vulnerabilities/j2ee/liferay-resource-leak.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/vulnerabilities/j2ee/liferay-resource-leak.yaml)

* Liferay Portal 7.3.0 GA1
* Liferay Portal 7.0.2 GA3

### 通过 JSON Web 服务导致的远程代码执行 (RCE)

* [nuclei-templates/http/cves/2020/CVE-2020-7961.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/cves/2020/CVE-2020-7961.yaml)

## 参考资料 (References)

* [Pentesting Liferay Applications - Víctor Fresco - February 6, 2025](https://www.tarlogic.com/blog/pentesting-liferay-applications/)
* [How to exploit Liferay CVE-2020-7961 : quick journey to PoC - Thomas Etrillard - March 30, 2020](https://www.synacktiv.com/en/publications/how-to-exploit-liferay-cve-2020-7961-quick-journey-to-poc.html)
