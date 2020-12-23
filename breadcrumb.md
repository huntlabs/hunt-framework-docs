## Breadcrumb(面包屑)

Hunt框架可以通过扩展ServiceProvider方式来满足项目构建面包屑模块的需求。主要实现步骤如下：

### 1. 新建一个面包屑的ServiceProvider

新建一个名为`MyBreadcrumbServiceProvider`的`ServiceProvider`，如：/source/providers/MyBreadcrumbServiceProvider.d

```d
module app.providers.MyBreadcrumbServiceProvider;

import hunt.framework;

class MyBreadcrumbServiceProvider : BreadcrumbServiceProvider 
{
    override void boot() 
    {
        breadcrumbs.register("sign1", (Breadcrumbs trail, Object[] params...) {
            trail.push("ROOT", url("index.testRoot"));
        });

        breadcrumbs.register("sign2", (Breadcrumbs trail, Object[] params...) {
            // 绑定上级节点的标识
            trail.parent("sign1");
            trail.push("CHILD", url("index.test"));
        });
    }
}
```

其中 breadcrumbs.register 是注册 Breadcrumb 的一个节点

结构说明如下:
- breadcrumbs.register 的第一个参数表示这个节点的唯一标识.
- trail.push 的参数分别为 页面显示的内容, 点击跳转的链接. 
- 当前节点非根节点时, 需要增加 trail.parent, 参数为父级别的节点的唯一标识.

### 2. 注册面包屑的ServiceProvider 

在 main() 中注册 MyBreadcrumbServiceProvider 使配置生效.

```d
import hunt.framework;
import app.providers.MyBreadcrumbServiceProvider;

void main(string[] args) 
{
    Application appInstance = Application.instance();
    appInstance.register!MyBreadcrumbServiceProvider;
    appInstance.run(args);
}
```

### 3. 在 Controller 里获取面包屑

创建一个 /source/app/controller/TestController.d 
并且创建两个方法分别输出 MyBreadcrumbServiceProvider.d 中设置的节点设置及关系。

```d
module app.controller.TestController;

import hunt.framework;
import std.conv;
import std.stdio;

class TestController : Controller
{

    mixin MakeController;

    @Action
    string testRoot()
    {
        BreadcrumbItem[] breadCrumbs = Application.instance.breadcrumbs().generate("sign1");
		string[] tmp;
		foreach (BreadcrumbItem one; breadCrumbs)
        {
            // BreadcrumbItem的数据转成字符串
            tmp ~= one.toString();
        }
        return tmp.to!string;
    }

    @Action
    string testChild()
    {
        BreadcrumbItem[] breadCrumbs = Application.instance.breadcrumbs().generate("sign2");
        string[] tmp;
		foreach (BreadcrumbItem one; breadCrumbs)
        {
            tmp ~= one.toString();
        }
        return tmp.to!string;
    }

}
```

### 4. 在路由文件中补充对应的路由信息

路由文件 `/config/routes` 中补充如下内容:

```ini
GET     /root           test.testRoot
GET     /child          test.testChild 
```

### 5. 编译运行查看结果

在本示例 MyBreadcrumbServiceProvider.d 绑定了 `sign1` 是根节点，testRoot 输出的内容只有节点自己的信息:

```
["title: ROOT, link: /root/"]
```

在本示例 MyBreadcrumbServiceProvider.d 绑定了 `sign2` 设置父级节点，testChild 输出的内容父级和节点自己的信息:

```
["title: ROOT, link: /root/", "title: CHILD, link: /child/"]
```

在实际使用中可以根据自己的需求输出 Breadcrumb 的内容:

可以使用 `title` 属性输出节点的显示内容, 只用 `link` 属性输出节点的链接. 例如:

```d
module app.controller.TestController;

import hunt.framework;
import std.conv;
import std.stdio;

class TestController : Controller
{

    mixin MakeController;

    /** 
     * 本示例中的输出结果为 : 
     *      1级节点 标题为: ROOT, 跳转url: /root/;
     */
    @Action
    string testRoot()
    {
        BreadcrumbItem[] breadCrumbs = Application.instance.breadcrumbs().generate("sign1");
		string tmp;
		foreach (long k, BreadcrumbItem one; breadCrumbs)
        {
            tmp ~= (k + 1).to!string ~ "级节点 标题为: " ~ one.title ~ ", 跳转url: " ~ one.link ~ "; ";
        }
        return tmp;
    }
    
    /** 
     * 本示例中的输出结果为 : 
     *      1级节点 标题为: ROOT, 跳转url: /root/; 2级节点 标题为: CHILD, 跳转url: /child/;
     */
    @Action
    string testChild()
    {
        BreadcrumbItem[] breadCrumbs = Application.instance.breadcrumbs().generate("sign2");
	    string tmp;
		foreach (long k, BreadcrumbItem one; breadCrumbs)
        {
            tmp ~= (k + 1).to!string ~ "级节点 标题为: " ~ one.title ~ ", 跳转url: " ~ one.link ~ "; ";
        }
        return tmp.to!string;
    }
}
```
