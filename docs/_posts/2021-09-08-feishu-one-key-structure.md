---
layout: post
title:  "一键创建飞书文档目录结构"
date:   2021-09-08 19:15:03 +0800
categories: tool
---
## 背景需求

1. 一直使用 [confluence](https://www.atlassian.com/software/confluence) 进行文档相关的管理，已经有类似如下的目录结构：

   ![image-20210908184503061](/assets/image-20210908184503061.png)

2. 由于公司策略调整，需要统一迁移到飞书文档。

3. confluence已经有脚本一键生成上述的目录结构，因此也需要写一个脚本实现类似的功能



## 解决思路

### 1. 寻找Open API

* 飞书是一个已经相对成熟的平台，一般都会有相应的open api支持第三方进行扩展，简单的google搜索一下，就找到如下的文档：
  * [如何使用文档的OpenAPI功能](https://www.feishu.cn/hc/zh-CN/articles/360049067877)
  * [开发指南](https://open.feishu.cn/document/ukTMukTMukTM/ukjM5YjL5ITO24SOykjN)

* 快速的过了文档内容，发现openAPI提供的功能应该是满足上述的要求。不过坑爹的事，同样也来了，调用接口，需要提供accessToken, 文档里也提供了相应的介绍说明 ：[API访问凭证概述](https://open.feishu.cn/document/ukTMukTMukTM/uMTNz4yM1MjLzUzM)

  >
  > 要获取带有授权的飞书开放平台访问凭证（access token），你需要先完成以下步骤：
  >
  > 1. 应用在开放平台上完成注册
  > 2. 在开发者后台声明所需要的权限，并发布一个应用的版本
  > 3. 应用的使用者（用户或者租户管理员）对应用授权

* 好家伙，上面3样东西，要去完成，实在太麻烦了，不符合我“`懒惰`”的人格，因此，先把这个方案**hold-on**



### 2. 模拟业面操作

* 作为一个工具型的打杂，通过脚本模拟页面操作，这是一个必备的技能
* 通过在页面上模拟操作，很快就找到了创建请求的示例：
  * ![image-20210908192913876](/assets/image-20210908192913876.png)
* 通过创建了一堆文档之后，很快找到其中的关键元素：
  * Url: https://xiaopeng.feishu.cn/space/api/explorer/create/
  * Method: Post
  * header: `refer`和 `cookie`
    * ![image-20210908190342534](/assets/image-20210908190342534.png)
  * body:
    * ![image-20210908190450591](/assets/image-20210908190450591.png)
* 补充上述无聊里关联的一些字段说明：
  * 针对`body`里的`parent_token`, 其实就代表在哪个`文件夹`里进行文件的创建，获取形式查看链接：[docToken](https://open.feishu.cn/document/ukTMukTMukTM/ukjM5YjL5ITO24SOykjN#2b507ee2)
  * cookie里的`session` 需要你在浏览器(e.g. chrome)里完成`飞书文档的登录`，然后可以通过如下的方式获取：
    * ![image-20210908191006800](/assets/image-20210908191006800.png)



## 编写脚本

* 有了上面的`模拟业面操作`找出来的请求方式，就可以很容易编写脚本了，基于java + Spring，提供参考例子如下 :

  * [TemplateCreator](https://github.com/nick-yegw/script_tool/blob/main/src/main/java/org/nick/tool/feishu/TemplateCreator.java)
    * 说明：
      * 请求参数中涉及的`type`代表需要创建文件的不同类型，已知如下：（`其他类型，麻烦自行探索`）
        * 0 - 文件夹
        * 2 - doc文档

  

## 后续跟进 - 坑爹的飞书

* 通过上述的脚本创建的`doc文档`，`首次`进入doc文档时，需要人工再次输入标题。

* 现象如下：

  1. 通过上述的脚本创建的`doc文档`, 虽然已经将`标题`传给接口，但是`首次进入doc文档`时，会出来如下的现状：标题没有自动同步
    * ![image-20210908191831417](/assets/image-20210908191831417.png)
  2. 更坑爹的情况出现了，如果当你在标题里输入`任意的字符`时，会自动将文档的标题进行了`同步`:
    * ![image-20210908192052101](/assets/image-20210908192052101.png)
  3. 如果`撤销输入`或者`删除标题`时，脚本里输入的标题就会跟你`say goodbye` （文档的自动保存） , 变成如下，需要你人工再次输入
    * ![image-20210908192400452](/assets/image-20210908192400452.png)

  

### 曾经的努力：

  * 参考`模拟业面操作`的经验，尝试模拟人工的首次输入标题，解决上述的问题，但是，这个时候`坑爹(短时间解决不了)的事`出现了，无论是在`标题`里输入文字，还是在`正文`里输入的文字，都是使用了同一个接口(`不知道定位对不对`)，而且没有明显的识别区分，示例如下：

      * ![image-20210908193304190](/assets/image-20210908193304190.png)

          * 左边是`正文`，右边是`标题`

        
