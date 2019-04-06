---
title: ArcEngine中打开各种数据源（WorkSpace）的连接
tags:
- ArcEngine
- IWorkspace
categories:
- ArcEngine
- 记录&&踩过的坑
---

## 1.创建workspacefactory
```C#{.line-numbers}
 //方式1
 Type factoryShpType = Type.GetTypeFromProgID("esriDataSourcesFile.ShapefileWorkspaceFactory");//Shp
 Type factoryGdbType = Type.GetTypeFromProgID("esriDataSourcesGDB.FileGDBWorkspaceFactory");//Gdb
 Type factorySdeType = Type.GetTypeFromProgID("esriDataSourcesGDB.SdeWorkspaceFactory");//Sde
 Type factoryMdbType = Type.GetTypeFromProgID("esriDataSourcesGDB.AccessWorkspaceFactory");//Mdb
 Type factorySqliteType = Type.GetTypeFromProgID("esriDataSourcesGDB.SqlWorkspaceFactory");//Sqlite
 Type factoryCadType = Type.GetTypeFromProgID("esriDataSourcesFile.CadWorkspaceFactory");//Cad
 IWorkspaceFactory workspaceFactory = (IWorkspaceFactory)Activator.CreateInstance(factory***Type);

 ------------------------------------------------------------------------------------------------------
 //方式2
 IWorkspaceFactory wksGdbFactory = new FileGDBWorkspaceFactoryClass();
 IWorkspaceFactory wksSdeFactory = new SdeWorkspaceFactoryClass();
 IWorkspaceFactory wksMdbFactory = new AccessWorkspaceFactoryClass();
 IWorkspaceFactory2 wksRasterFactory = new RasterWorkspaceFactoryClass();
```
## 2.调用workspacefactory的方法打开数据源
```C#{.line-numbers}
 //MDB,GDB,SDE文件路径
 string strDbPath = "文件路径";
 IWorkspace wks = wksFactory.OpenFromFile(strDbPath, 0);

 //SDE也可通过连接信息打开
 IPropertySet propSet=new PropertySetClass();
 propSet.SetProperty("server","服务器机器名" );
 propSet.SetProperty("instance","SDE运行的端口号");
 propSet.SetProperty("user","用户名");
 propSet.SetProperty("password","口令" );
 propSet.SetProperty("password","口令" );
 IWorkspace wks = wksFactory.Open(propSet, 0);
```
PS：获取MDB，GDB文件路径时要注意，“右键数据库文件=》属性=》安全=》对象名称”，此处的文件路径有问题，如要选路径可从资源管理器的文件路径复制。
string connectionString = string.Format("Provider=ESRI.GeoDB.OleDB.1;Data Source={0};Extended Properties=workspacetype=esriDataSourcesGDB.FileGDBWorkspaceFactory.1;Geometry=WKB", strDbPath);