# Views

- [Creating Views](#creating-views)
- [Passing Data To Views](#passing-data-to-views)
- [Optimizing Views](#optimizing-views)

<a name="creating-views"></a>
## Creating Views

Views contain the HTML served by your application and separate your controller / application logic from your presentation logic. Views are stored in the `views` directory. A simple view might look something like this:

```html
    <!-- View stored in views/greeting.html -->

    <html>
        <body>
            <h1>Hello, {{ name }}</h1>
        </body>
    </html>
```

Since this view is stored at `views/greeting.html`, we may return it using the global `view` helper like so:

```D
module  app.controller.IndexController;

import  hunt.framework;

class IndexController : Controller
{
    mixin MakeController;

    @Action
    string index()
    {
        view.assign("name", "James");

        return view.render("greeting");
    }
}
```

As you can see, the first argument passed to the `view` helper corresponds to the name of the view file in the `views` directory. The second argument is an array of data that should be made available to the view. In this case, we are passing the `name` variable, which is displayed in the view like [Twig and Jinja2 syntax](https://github.com/huntlabs/hunt-framework/wiki/View).

<a name="passing-data-to-views"></a>
## Passing Data To Views

As you saw in the previous examples, you may pass an array of data to views:
```D
    view.assign("greetings", ["name" => "Victoria"]);
```

When passing information in this manner, the data should be an array with key / value pairs. Inside your view, you can then access each value using its corresponding key, such as `{{ greetings.name }}`.

<a name="optimizing-views"></a>
## Optimizing Views

By default, views are compiled on demand. When a request is executed that renders a view, hunt-framework will determine if a compiled version of the view exists. If the file exists, hunt-framework will then determine if the uncompiled view has been modified more recently than the compiled view. If the compiled view either does not exist, or the uncompiled view has been modified, hunt-framework will recompile the view.

> {tip} More item value reference file [hunt-framework/wiki](https://github.com/huntlabs/hunt-framework/wiki/View)