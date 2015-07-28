title: 嵌入式tomcat与JFinal集成
---
JFinal自带jetty作为开发工具，部署时就要重新处理，无法生成jar，本次直接集成嵌入式tomcat，可以生成jar文件，运行。
未处理改动class文件时，自动重载功能！

项目结构：
demo
  |--src
  	  |--main
    	  |--java
		  |--resources
	   |--test
	      |--java
		  |--resources
   |--webapp

启动类如下：
```java
public class Application {

	public static void main(String[] args) {
		Tomcat tomcat = new Tomcat();
		tomcat.setPort(8080);
		
		try {
			String prefix = System.getProperty("user.dir");
			StandardServer server = (StandardServer) tomcat.getServer();
			server.addLifecycleListener(new AprLifecycleListener());
			Context context = tomcat.addWebapp(contextName, new File(prefix + "//webapp").getAbsolutePath());
			
			initJfinal(context);
			
			tomcat.start();
			server.await();
		} catch (Exception e) {
			System.err.println("启动服务器错误！");
			e.printStackTrace();
			try {
				tomcat.stop();
			} catch (LifecycleException e1) {
				e1.printStackTrace();
			}
		} 
		
	}
	
	private static void initJfinal(final Context context){
		//初始化filter
		FilterDef jfinalFilter = new FilterDef();
		jfinalFilter.setFilterName("JFinalFilter");
		jfinalFilter.setFilterClass(JFinalFilter.class.getName());
		jfinalFilter.addInitParameter("exclusions", "/assets/*,*.js,*.css,*/druid*,/upload/*");
		jfinalFilter.addInitParameter("configClass", LiteERPConfig.class.getName());
		FilterMap fmap = new FilterMap();
		fmap.setFilterName("JFinalFilter");
		fmap.addURLPattern("/*");
		context.addFilterDef(jfinalFilter);
		context.addFilterMap(fmap);
	}
	
}

```