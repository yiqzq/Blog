# SpringMVC使用自定义视图以及视图解析器

[toc]

​	我们知道在获得了`ModelAndView`(简称mv)对象之后,前端控制器会根据这个对象通过`ViewResolver`解析出一个view对象,最后通过view对象渲染整个页面,传递给用户。

其中系统默认的是`InternalResourceViewResolver`,当然我们也可以通过实现接口来自定义`viewResolver`。

1. 创建`MyViewReslover`类，并实现`ViewResolver`（想要自定义类型，必须实现），`Ordered`（这个是为了接下来的排序，下面会说）接口。

   - `ViewResolver`接口下有一个方法，`public View resolveViewName(String viewName, Locale locale) `,作用是通过传入的`viewName`，看当前的`ViewResolver`是否可以解析，如果可以解析，需要返回一个对应的`View`对象。

   - `Ordered`接口下面有一个方法，`public int getOrder()`,用于返回一个int类型的值，作用是比较。因为在SpringMVC的底层中,有下面一段代码

     ```java
     for (ViewResolver viewResolver : this.viewResolvers) {
     			View view = viewResolver.resolveViewName(viewName, locale);
     			if (view != null) {
     				return view;
     			}
     		}
     ```

     ​		大概解释一下，`SpringMVC`允许有多个视图解析器对`viewName`进行解析,代码中的`viewResolvers`是一个`List`容器，·`SpringMVC`会遍历其中的视图解析器，那么这就存在一个先后进行解析的顺序，这就是`Orderd`的作用，方便认为控制顺序，因为在代码中我们可以知道，一旦找到一个匹配的视图解析器，就会直接返回`View`对象，`List`中剩下的就不会在看了。所以，为了认为控制这个`List`中的先后解析顺序，引入了这个接口。

     ​		**要注意的是，值越小，优先级越高。**

```java
@Component
public class MyViewReslover implements ViewResolver, Ordered {
    @Override
    public View resolveViewName(String viewName, Locale locale) throws Exception {
        if (viewName.startsWith("yiqzq:")) {
            return new MyView();
        } else {
            return null;
        }
    }

    @Override
    public int getOrder() {
        return 0;
    }
}

```

2. 创建`MyView`类，并实现`View`接口。

- View接口下有两个方法

```java
String getContentType();//获取当前view的ContentType()
void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;//允许使用model里的数据来渲染页面

```



```java
@Component
public class MyView implements View {

    @Override
    public String getContentType() {
        return "text/html;";
    }

    @Override
    public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter pw = response.getWriter();
        pw.write("<h1>这是我的自定义视图</h1>");
        List<String>fruits= (List<String>) model.get("fruit");
        for(int i=0;i<fruits.size();i++){
            pw.write("<h3>"+fruits.get(i)+"</h3>");
        }
    }
}

```

3. 创建控制器

```java
@Controller
public class MyViewResolverController {
    @RequestMapping("/myview")
    public String handle(Model model) {
        List<String> fruit = new ArrayList<>();
        fruit.add("apple");
        fruit.add("banana");
        fruit.add("orange");
        model.addAttribute("fruit", fruit);
        return "yiqzq:/myview";
    }
}

```

4. 运行即可

可以看到这是自己写的视图

![image-20200302185336272](C:\Users\39268\AppData\Roaming\Typora\typora-user-images\image-20200302185336272.png)