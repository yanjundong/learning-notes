# 服务启动流程

1. 检查集群是否正常启动；
2. 启动 nacos（`192.168.30.5`的 `/opt/software/`大概）命令：`sh startup.sh -m standalone`；
3. 使用`start.sh`脚本启动`gis-gateway.jar` `gis.jar` `gis-management.jar`，(多台服务器)
4. 使用`start.sh`脚本启动 `gis-inout.jar`（一台服务器，）
5. 启动`30.5` 服务器上的 tomcat，路径大概也是`/opt/software`

# 显示数据

显示，点击“正射影像瓦片”，出现二级菜单，
再点击“city”，出现金字塔数据，右键“记载金字塔”即可。

# 入库数据

## 数据格式类型

```tex
vectormap("电子地图瓦片"),

dem("DEM瓦片"),

shadingmap("晕渲地图瓦片"),

dom("正射影像瓦片"),

vector("矢量数据瓦片"),

gazetteer("地名瓦片");
```

## 文件名称

`数据格式类型_金字塔二级目录_金字塔名称 `，例如`dom_city_beijing`

存放格式：

`/tiles/jiwei/dom_ledong_checkData/`

```tex
     * dom_city_beijing
     *      Export
     *          template
     *              template-Export
     *                  L0
     *                      R0
     *                          C0.png
     *                  L1
     *                  ...
     *
     *                  L19
     *              template.xml
```





![image-20210618171444022](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210618171444022.png)

## 操作

1. 点击右上角”数据管理“，再点击”产品数据和入库“；

2. 表单“数据格式”选择`保障库格式标准`；

3. 把数据存放路径复制到`文件路径`；

4. 点击“开始入库”

5. 在“入库进度”中查看进度

   ![image-20210618172726822](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210618172726822.png)

6. 100%后点击“完成”。

# 星环集群更换许可

![image-20210618172853478](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210618172853478.png)

![image-20210618173142855](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210618173142855.png)

# 项目结构

## front-server

前置机

- MySQL
- MyBatis

## gis

除了`cloud/inout`，其余都是李老师设计的对外暴露的接口。

## gis-common

公共内容

## gis-gateway

![image-20210618174917631](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210618174917631.png)

## gisManagement

包装了gis，主要服务于前端

## 看代码

1. gis-common
2. gis
3. gisManagement
4. gis-gateway（瞄一眼）
5. front-server（稍微看看就行）

# 学习内容

1. SpringCloud
2. Hadoop
3. HBase

