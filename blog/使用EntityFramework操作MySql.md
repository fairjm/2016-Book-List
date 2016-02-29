本文来自 fairjm@ituring.com.cn  
转截请注明出处    

---  

本来想用F#的....结果失败了 去爆栈提了个问题...暂时没人解答...所以还是用C#吧...哎   
为什么要连MySql,因为Sql Server开箱即用,而且工作中用的是mysql,取数据分析用ADO.NET有点累,一切面向偷懒.

框架使用EF,版本用最新的,一步一步往下做.  
因为就是个示例项目所以用good old的`Console Application`就行啦.  
空项目如下:  

  
添加依赖,右击`References`,选择`Manage NuGet Packages`,选择browse.搜索`EntityFramework`和`Mysql.Data.Entity`,全部选最新版本.  一路点Yes,accept就好了.

此时package的依赖变成了这样:  

    <?xml version="1.0" encoding="utf-8"?>
    <packages>
      <package id="EntityFramework" version="6.1.3" targetFramework="net452" />
      <package id="MySql.Data" version="6.9.8" targetFramework="net452" />
      <package id="MySql.Data.Entity" version="6.9.8" targetFramework="net452" />
    </packages>

App.config也变得丰盈了起来:  

    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <configSections>
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
      </configSections>
      <startup>
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
      </startup>
      <entityFramework>
        <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
          <parameters>
            <parameter value="mssqllocaldb" />
          </parameters>
        </defaultConnectionFactory>
        <providers>
          <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
          <provider invariantName="MySql.Data.MySqlClient" type="MySql.Data.MySqlClient.MySqlProviderServices, MySql.Data.Entity.EF6, Version=6.9.8.0, Culture=neutral, PublicKeyToken=c5687fc88969c44d">
          </provider>
        </providers>
      </entityFramework>
      <system.data>
        <DbProviderFactories>
          <remove invariant="MySql.Data.MySqlClient" />
          <add name="MySQL Data Provider" invariant="MySql.Data.MySqlClient" description=".Net Framework Data Provider for MySQL" type="MySql.Data.MySqlClient.MySqlClientFactory, MySql.Data, Version=6.9.8.0, Culture=neutral, PublicKeyToken=c5687fc88969c44d" />
        </DbProviderFactories>
      </system.data>
    </configuration>  

依赖搞定,接下来添加实体对象,这里就不手工写了,手工写无非是将连接字符串配在App.config下,自己继承下DbContext,手写实体类.太累.还是那句话,面向懒惰编程.  

右击项目,add -> new item -> data -> ADO.NET Entity Data Model  
名字自己取一个  
然后按以下步骤添加:  
![enter image description here][2]
![enter image description here][3]  
关于Port,请点击上图中的Advanced,在里面找到Port选项.  
按ok之后.  

如果你用的依赖项和我一样,而且装mysql的时候还装了.NET的连接,那肯定就是直接退出了.对,你没看错,直接退出.  
  
把Nuget的包变成和本地版本一样(自己导入引用或者去mysql官网下最新的 那个Mysql Installer不能升级到最新的不知道为什么),不然点next依旧闪退.  
接下来看到了令人激动的这个界面:
![enter image description here][4]  
当点击完成的时候,WTF:  
![enter image description here][5]  

这其实是个bug,mysql的bug... ...解决方法很简单,要把mysql的一个优化项给关了.  
http://stackoverflow.com/questions/33575109/mysql-entity-the-value-for-column-isprimarykey-in-table-tabledetails-is  

> 1. Open Services (services.msc) and restart MySQL57 service.
2. Execute the following commands in MySQL.
   use database_name;  
   set global optimizer_switch='derived_merge=OFF';
3. Update the .edmx.    

接下来就好了...可以看到Designer了  
![enter image description here][6]

接下来就可以愉快地写代码啦~~~  
![enter image description here][7]  
  
注意如果提示初始化错误,检查一下DbContext里的初始化参数的值和app.config的连接字符串配置是否相同  
![enter image description here][8]    

其实整个过程是比较简单的.  
这里有几点注意  
**本地如果装了.NET驱动,要注意和依赖包的版本保持一致**  
**那个坑爹的bug**  

code first也比较简单.  
但也有几点要注意,这里有点晚了,我懒得写具体过程了.....  
**一定要从空的数据库初始化,不要有表,不要有表,不要有表...** 不然会发生奇怪的事的,是什么奇怪的事,我不说,你们猜.  
**请把`[DbConfigurationType(typeof(MySqlEFConfiguration))]`写在DbContext子类上(或者改配置文件)** 这一项是code first生成的代码里没有的.  
然后就可以愉快地commit和update数据库了.  

本文终结

---  
本文来自 fairjm@ituring.com.cn  
转截请注明出处    
