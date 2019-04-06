---
title: 通过Arcpy发布地图服务
tags:
- ArcEngine
categories:
- ArcEngine
- Arcpy
---


## 1.发布地图服务的流程
使用 ArcPy 将地图文档自动发布到 GIS 服务器的流程分为四步：
* 第一步，运行 CreateMapSDDraft 函数。CreateMapSDDraft 的输出是服务定义草稿 (.sddraft) 文件，服务定义草稿由地图文档、服务器信息和一组服务属性组合而成。
* 第二步，使用 AnalyzeForSD 函数分析输出的服务定义草稿文件的适用性和潜在性能问题。
* 第三步，使用 Stage Service 地理处理工具将服务定义草稿转换为完全合并的服务定义 (.sd) 文件。过渡过程会编译成功发布 GIS 资源所需的所有必要信息。如果选择将数据复制到服务器，则将在服务定义草稿阶段添加数据。
* 最后，使用 上载服务定义地理处理工具上载服务定义文件并将其作为 GIS 服务发布到特定的 GIS 服务器。此步骤将获取服务定义文件、将其复制到服务器、提取所需信息并发布 GIS 资源。

## 2.调用函数参数详解
####第一步：创建草图文件
CreateMapSDDraft (map_document, out_sddraft, service_name, {server_type}, {connection_file_path}, {copy_data_to_server}, {folder_name}, {summary}, {tags})

|参数|说明|类型
|:-------|:-------|:-------
|map_document|Map Document 类型的对象，即一个mxd文档。|Map Document
|out_sddraft|Service Definition Draft (.sddraft) 文件输出路径。|String
|service_name|服务的名字，由字母和数字组成，不允许使用空格或特殊字符，长度不得超过120。|String
|server_type|服务的类型，如果未提供「connection_file_path」参数，则必须提供「server_type」。如果提供了「connection_file_path」参数，则从连接文件中获取「 server_type」。<br> • ARCGIS_SERVER — ArcGIS for Server 服务类型，默认值。<br> • FROM_CONNECTION_FILE — 从 connection_file_path 参数获取服务类型。<br> • SPATIAL_DATA_SERVER — Spatial Data Server 服务类型，ArcGIS 10.2.1 版本之后就不再支持。<br> • MY_HOSTED_SERVICES — My Hosted Services 服务类型，应用与 ArcGIS Online 或者 Portal for ArcGIS 的托管服务。|String
|connection_file_path|ArcGIS for Server connection file (.ags) 文件的路径。通常在ArcCatalog创建后的路径为「C:\Users\Administrator\AppData\Roaming\ESRI\Desktop10.2\ArcCatalog」|String
|copy_data_to_server|mxd文档的数据是否要拷贝到服务器中。当数据没有在服务器内被注册时，此参数应设为false，反之应设为true。<br> 当「server_type」设置为SPATIAL_DATA_SERVER时，「copy_data_to_server」将始终为False。 Spatial Data Server 服务始终使用已注册的数据，因此不会将数据复制到服务器。<br> 当「server_type」设置为MY_HOSTED_SERVICES时，「copy_data_to_server」将始终为True。My Hosted Maps services 服务始终将数据复制到服务器。|Boolean
|folder_name|服务发布的文件夹名，如果不存在则会新建，默认值None对应的是根文件夹。|String
|summary|服务的摘要。|String
|tags|服务的标签。|String

####第二、三、四步
可以直接在arcgis帮助文档内查看，有中文的。

## 3.实现代码
下面是一个发布服务的例子，实现的功能是遍历一个文件夹，将文件夹内.mxd结尾的文件都发布上服务器。
```python{.line-numbers}
# -*- coding: utf-8 -*-
import arcpy
import os
import xml.dom.minidom as DOM

def SetSddraftParam(sddraft_file_path):
    '''修改Sddraft文件的参数。（一般就修改开启WMS、WFS功能）

    Args:
        sddraft_file_path: .sddraft文件的路径。
    '''
    doc = DOM.parse(sddraft_file_path)
    ext = doc.getElementsByTagName('Extensions')[0]
    svcExts = ext.childNodes
    for svcExt in svcExts:
        typename_ele = svcExt.getElementsByTagName('TypeName')[0]
        if typename_ele.firstChild.data == 'WMSServer':
            enable_ele = svcExt.getElementsByTagName('Enabled')[0]
            enable_ele.firstChild.data = 'true'
            break
    if os.path.exists(sddraft_file_path):
        os.remove(sddraft_file_path)
    f = open(sddraft_file_path, 'w')
    doc.writexml(f)
    f.close()

def GetAGSConnectionFile(out_folder_path):
    '''在指定文件夹新建test.ags文件。

    Args:
        out_folder_path: .ags文件的输出文件夹路径。

    Returns:
        返回.ags文件的路径。
    '''
    out_name = 'test.ags'
    server_url = 'http://localhost:6080/arcgis/admin'
    use_arcgis_desktop_staging_folder = False
    staging_folder_path = out_folder_path
    username = 'siteadmin'
    password = '123456'
    out_file_path = os.path.join(out_folder_path, out_name)
    if os.path.exists(out_file_path):
        os.remove(out_file_path)
    arcpy.mapping.CreateGISServerConnectionFile('ADMINISTER_GIS_SERVICES', 
                                                out_folder_path,
                                                out_name,
                                                server_url,
                                                'ARCGIS_SERVER',
                                                use_arcgis_desktop_staging_folder,
                                                staging_folder_path,
                                                username,
                                                password,
                                                True)
    return out_file_path

def PublishMxd(mxd_file_path, mxd_folder_path, con_file_path):
    '''发布服务。

    Args:
        mxd_file_path:mxd文档的路径。
        mxd_folder_path:mxd文档所在文件夹的路径。
        con_file_path:服务器连接文件路径
    '''
     #检查mxd文件是否存在
    print "Checking mxd file path..."
    if os.path.exists(mxd_file_path) == False:
        print "mxd file is not exist！"
        return
    
    # 打开mxd文档
    try:
        print "Opening mxd file..."
        mxd = arcpy.mapping.MapDocument(mxd_file_path)
    except Exception, e:
        print "open mxd error: ", e
        return

    # 获取默认的数据框
    print "Loading mxd file default dataframes..."
    df = ""
    try:
        frames = arcpy.mapping.ListDataFrames(mxd, "图层")
        if len(frames) == 0:
           frames = arcpy.mapping.ListDataFrames(mxd, "Layers") 
        df = frames[0]
    except Exception, e:
        print "load mxd file default dataframes failed：", e
        return
    # 组织参数发布服务
    mxdNameWithExt = os.path.basename(mxd_file_path)
    (serviceName, extension) = os.path.splitext(mxdNameWithExt)
    sddraft_file_path = os.path.join(mxd_folder_path, serviceName + '.sddraft')
    summary = 'Test'
    tags = 'Test'
    # 创建草图文件
    print "CreateMapSDDraft..."
    if os.path.exists(sddraft_file_path):
        os.remove(sddraft_file_path)
    arcpy.mapping.CreateMapSDDraft(mxd, sddraft_file_path, serviceName, 'ARCGIS_SERVER', con_file_path, False, None, summary, tags)
    # 设置草图文件内的参数（开启WMS功能，WFS功能等）
    SetSddraftParam(sddraft_file_path)
    # 分析草图文件
    analysis = arcpy.mapping.AnalyzeForSD(sddraft_file_path)
    if analysis['errors'] == {}:
        for message in analysis['messages']:
            print analysis['messages'][message]
            print message[0].encode("gb2312") + "(%s)" % message[1]
        for warning in analysis['warnings']:
            print analysis['warnings'][warning]
            print warning[0].encode("gb2312") + "(%s)" % warning[1]
        # 过渡服务
        print "StageService..."
        sdPath = os.path.join(mxd_folder_path1, serviceName + '.sd')
        arcpy.StageService_server(sddraft_file_path, sdPath)
        # 上传服务定义
        print "UploadServiceDefinition_server..."
        arcpy.UploadServiceDefinition_server(sdPath, con_file_path)
    else:
        for error in analysis['errors']:
            print analysis['errors'][error]
            print error[0].encode("gb2312") + "(%s)" % error[1]

def PublishAll(mxd_folder_path):
    '''遍历指定文件夹内的所有mxd文档，并逐个发布服务。

    Args:
        mxd_folder_path:包含mxd文档的文件夹路径。
    '''
    print "Check folder path..."
    if os.path.isdir(mxd_folder_path) == False:
        print "folder path is not exist！"
        return
    print "Get .ags file..."
    con_file_path = GetAGSConnectionFile(mxd_folder_path)
    print "******************Traversing a folder******************"
    files = os.listdir(mxd_folder_path)
    mxdCount = 0
    for f in files:
        if f.endswith(".mxd"):
            mxdCount = mxdCount + 1
    mxdNo = 1
    for f in files:
        if f.endswith(".mxd"):
            mxd_file_path = os.path.join(mxd_folder_path, f)
            print "Publishing: " + f + "(%d/%d)" % (mxdNo, mxdCount)
            mxdNo = mxdNo + 1
            PublishMxd(mxd_file_path, mxd_folder_path, con_file_path)
        else:
            continue

mxd_folder_path = r'E:\MyCode\Mypy\py2\mxd'
publishServices.PublishAll(mxd_folder_path)
```

##4.备注
针对上面的内容还有以下几点需要说明：
1. mxd文件的名称不能是中文，python2.7对中文的支持不友好，总会碰到问题，所以都用英文比较好。
2. 通过arcmap界面发布服务时设置的参数可以在sddraft草图文件内找到，并通过修改文件配置来设置CreateMapSDDraft函数没囊括的参数。另外对没有函数可以修改已发布的服务参数问题，设想可以通过重复发布服务覆盖来实现需求，但是暂时还没有尝试。
3. 切片缓存功能还没添加。
