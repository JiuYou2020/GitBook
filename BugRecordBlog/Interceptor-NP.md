原因：不同于Filter，interceptor依赖于Spring，请求在经过Filter之后才会经过interceptor，并且只能拦截controller或者static静态资源，它的底层原理是动态代理+反射。

在Spring项目中，它的初始化时机要早于Spring Context，因此，如果需要在其中注入Bean，推荐构造方法注入（推荐）或者使用`if`语句在需要使用的地方进行判断。