## Breadcrumbs

In `Hunt Framework`, you can use `Breadcrumb` by extending the `ServiceProvider`.

### 1. Create a `ServiceProvider` for your Breadcrumb

For example, you can name it as `MyBreadcrumbServiceProvider` in `/source/providers/MyBreadcrumbServiceProvider.d`.

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
            // Binding to the parent node
            trail.parent("sign1");
            trail.push("CHILD", url("index.test"));
        });
    }
}
```

The method `breadcrumbs.register()` is used to register a breadcrumb node.

**Node:**
- The first parameter of `breadcrumbs.register()` is a unique tag for the current node.
- The parameters of `trail.push()` are used for the content of page and the link. 
- When the current node is a non-root one, it is need to add a parameter `trail.parent` as the parent's tag.

### 2. Register the ServiceProvider 

The `MyBreadcrumbServiceProvider` should be registered in `main()`.

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

### 3. Get the Breadcrumbs in Controller

Create a Controller, for example `/source/app/controller/TestController.d`, and add tow methods to retrieve the all the nodes defined in MyBreadcrumbServiceProvider.d.

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

### 4. Define the all routes in config

Open the routes config file `/config/routes`, and add these:

```ini
GET     /root           test.testRoot
GET     /child          test.testChild 
```

### 5. The output

The `sign1` defined in the `MyBreadcrumbServiceProvider` is a root one, so the output is only itself:

```
["title: ROOT, link: /root/"]
```

The `sign1` defined in the `MyBreadcrumbServiceProvider` is a child one, so the output includes itself and it's parent:

```
["title: ROOT, link: /root/", "title: CHILD, link: /child/"]
```

Of course, you can get the the node's attributs. The `title` is for the title, and the `link` is for the url.  For example:

```d
module app.controller.TestController;

import hunt.framework;
import std.conv;
import std.stdio;

class TestController : Controller
{

    mixin MakeController;

    /** 
     * The output: 
     *      Level 1, the title: ROOT, the url: /root/;
     */
    @Action
    string testRoot()
    {
        BreadcrumbItem[] breadCrumbs = Application.instance.breadcrumbs().generate("sign1");
		string tmp;
		foreach (long k, BreadcrumbItem one; breadCrumbs)
        {
            tmp ~= "Level " ~ (k + 1).to!string ~ ", the title: " ~ one.title ~ ", the url: " ~ one.link ~ "; ";
        }
        return tmp;
    }
    
    /** 
     * The output: 
     *      Level 1, the title: ROOT, the url: /root/; Level 2, the title: CHILD, the url: /child/;
     */
    @Action
    string testChild()
    {
        BreadcrumbItem[] breadCrumbs = Application.instance.breadcrumbs().generate("sign2");
	    string tmp;
		foreach (long k, BreadcrumbItem one; breadCrumbs)
        {
            tmp ~= "Level " ~ (k + 1).to!string ~ ", the title: " ~ one.title ~ ", the url: " ~ one.link ~ "; ";
        }
        return tmp.to!string;
    }
}
```


### 6. To use in View

Binding all the breadcrumbs with a data model defined in a `View` as `breadcrumbs`. 

```d
class TestController : Controller
{
    
    @Action
    string showView() 
    {
        auto breadCrumbs = app().breadcrumbs().generate("sign");
        view.assign("breadcrumbs", breadCrumbs);
        
		return view.render("BreadCrumbsDemo");
    }
}
```

Render all the breadcrumbs in the page:

```html
{% if breadcrumbs.defined and breadcrumbs.length>0 %}
    <div class="col-sm mb-2 mb-sm-0">
        <nav aria-label="breadcrumb">
            <ol class="breadcrumb breadcrumb-no-gutter">
                {% for item in breadcrumbs %}
                    {% if not loop.last %}
                        <li class="breadcrumb-item">
                            {% if item.link %}
                                <a class="breadcrumb-link" href="{{ item.link }}">{{ item.title }}</a>
                            {% else %}
                                {{ item.title }}
                            {% endif %}
                        </li>
                    {% else %}
                        <li class="breadcrumb-item active" aria-current="page">{{ item.title }}</li>
                    {% endif %}
                {% endfor %}
            </ol>
        </nav>
    </div>
{% endif %}
```