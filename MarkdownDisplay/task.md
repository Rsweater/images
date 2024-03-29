# 郑州市建成区热岛效应时空演变特征

## 申请书内容

​  a. 郑州市建成区热岛效应时空演变特征。基于2013~2019年度Landsat8数据，进行辐射定标、大气矫正，使用RS图像处理软件ENVI单窗算法计算像元亮度温度值，制作地表温度反演图，<u>**结合历年气象站点温度观测数据**</u>，对反演结果进行验正；导入ArcGIS平台进行统计计算，将历年图像分别按照均值标准差方式进行重分类操作，分为**低温、次低温、中温、次高温、高温**5个类别，确定热岛强度指数；

<!--more-->

## 大致流程

1. 下载 `Landsat 8` 数据；
2. ~~去云处理；~~
3. 地表温度反演；
4. 结合气象站数据验证并修正反演结果(没数据)；
5. 利用 `ArcGIS` 进行裁剪、重分类、确定热岛强度。

### 下载 `Landsat-8 TIRS\OLSI`

- 尽量选 云指数 小的数据，主要还是选择郑州行政区内云量小的（可以直接肉眼看）。
- 下载方式有 [地理数据空间云](http://www.gscloud.cn/)、[USGS](http://earthexplorer.usgs.gov/)、[Landsat 8下载共享系统](http://ids.ceode.ac.cn/query.html)；

#### 地理空间数据云

**网址：**<http://www.gscloud.cn/>

**流程：**登录 --> 高级检索 --> 选数据集（Landsat 8 OLI_TIRS卫星数据产品) --> 选区域时间(河南郑州) --> 点选中间那一景 --> 找云量指数小的

**GIF演示：** ![gscloud_download_demo](https://cdn.jsdelivr.net/gh/Rsweater/images/img/gscloud_download_demo.gif)

**注意：**

1. 数据云选择郑州之后，会展示所有有拍到郑州行政区内的影像，中间那景可以全覆盖，只需要这一景即可
2. 这个平台很多不能预览，没办法更好的选择郑州区域内云量少（有些云量多，但是不在郑州区域内），可以结合USGS预览，地理空间数据云下载。

#### USGS

**网址：**<http://earthexplorer.usgs.gov/>

**流程：** 

1. `Enter Search Criteria`, 其中要设置 `研究区(郑州)`、`时间` 。

    - **研究区**：点选 `World Features` 之后，在Feature Name 中填入 `zhengzhou` ，点击 `Show` , 最后点选第一个 `Zhengzhou` 。

    选过研究区、时间之后点击 `Data Sets` 选择数据集，选过研究区和时间之后的截图：

2. `Data Sets`:  Landsat --> Landsat Collection 1 -->  Landsat Collection 1 Level-1 --> Landsat 8 LOI/TIRS C1 Level-1 --> Results.

3. Results. 

**Gif演示：** ![usgs_download](https://cdn.jsdelivr.net/gh/Rsweater/images/img/usgs_download.gif)

**注意：**

1. 网速慢、需要梯子；
2. 时间筛选的后面有云量筛选，建议不要用，有些可能云量大，但是郑州行政区内云相比较是最少的。真的太多，可以找6月或者9月的。（数据有限，16天一景，云会影响地表温度，会给最后重分类造成麻烦）
3. 有云掩膜产品，点击下载之后的选项卡里面，倒数第二个就是云掩膜产品。

### 利用 ENVI 进地表温度反演

​ 使用 [XXIN'S  Blog](https://www.ixxin.cn/2017/10/29/envi_landsat8_lst_sw/) 发布的插件做地表温度反演。（插件有还没装的私聊我）

**需要参数：**多光谱数据、热红外数据、大气透过率（大气水汽数据）、大气平均作用温度（计算高程）

![image-20200825220812775](https://cdn.jsdelivr.net/gh/Rsweater/images/img/image-20200825220812775.png)

**作者视频：** [B站搜索“温度反演”找到作者为“wudixinxin”的视频](https://www.bilibili.com/video/BV1F741167yX?p=6)

**流程+Gif演示：**

1. 打开`Landsat-8`数据集：File --> Open As --> Landsat -- GeoTIFF  with Metadata --> 选中 "_MTL.txt" 文件![open_date_landsat-8](https://cdn.jsdelivr.net/gh/Rsweater/images/img/open_date_landsat-8.gif)

2. 打开`landsat8 LST SingleWindow`进行地表温度反演：`Extensions --> Landsat8 LST SingleWindow`，其中

    1. 多光谱数据(**Multispectral Raster**) 就是第一个(名字以`MultiSpectral`结尾)
    2. 热红外数据(**Brightness Raster**) 就是第四个(名字以`Thermal`结尾)
    3. 大气透过率(**Atmospheric Water**) 通过 [NASA](https://atmcorr.gsfc.nasa.gov/) 网站查表得到
    4. 大气平均作用温度(**Atmospheric temperature**) 可以 [NASA](https://atmcorr.gsfc.nasa.gov/) 查表（需要借助envi的高程数据计算影像高程）或者直接百度当时的温度

    **打开并选择多光谱与红外数据GIF:**  

    ![lst_xinxin_open](https://cdn.jsdelivr.net/gh/Rsweater/images/img/lst_xinxin_open.gif)

    **`NASA`查表读取大气透过率获取大气平均作用温度表**

    ![nasa_temp_ssss](https://cdn.jsdelivr.net/gh/Rsweater/images/img/nasa_temp_ssss.gif)

    **注：**需要知道影像**拍摄时间**还有**中心经纬**(这个都一样，Latitude: 34.61083、Longitude：113.4636)

    **计算影像高程GIF(每幅影像应该一样，可以都做一下看看)**

    **步骤：**

    1. 打开世界高程数据：File --> Open World Data -- > Elevation; 
    2. 选择统计工具，Statistics --> Compute Statistics;
    3. 对待反演的数据集进行高程统计，将该数据集当作世界高程数据的子数据集去计算高成影像中该区域的平均高程![envi_gaocheng](https://cdn.jsdelivr.net/gh/Rsweater/images/img/envi_gaocheng.gif)

    **最后**补填上 大气透过率、平均作用温度、点选大气反演类型就ok了。

    **注意：**

    1. 上面括号内黑色英文不是对应的英文，而是插件对应的位置，因为也可以用其他相关的参数代替；
    2. 大气透过率要以符号 "-" 开头，正值表示使用的是大气水汽产品
    3. Atmospheric Type 跟 FLASHH 大气校正时的气溶胶类型是一个意思

### 利用 `ArcGIS` 进行裁剪、重分类、确定热岛强度

#### a. 裁剪

**流程：** System Toolboxes --> Spatial Analyst Tools.tbx --> Extraction --> Extract by Mask，进去之后有三个参数需要填 `Input raster`、 `Input raster or feature  mask data`、 `Output raster`，分别对应要裁剪的图像、保留的区域（也就是我们的行政区矢量图）、输出路径。

![image-20200826085452766](https://cdn.jsdelivr.net/gh/Rsweater/images/img/image-20200826085452766.png)

**注意：**

1. 这一步最好不要少，特别是对郑州行政区外有厚云的数据（会影响最小值，进而影响城市热岛指数），例如：![image-20200826085907150](https://cdn.jsdelivr.net/gh/Rsweater/images/img/image-20200826085907150.png)
2. 进行裁剪时，注意使用同一投影坐标系。我们下载的遥感影像应该都是`EPSG:32649`(WGS 84 / UTM zone 49N)，这里有`EPSG:32649`的郑州的shp[百度网盘，提取码0911](https://pan.baidu.com/s/1CHu8wwY4BPrRk_vIar-qdg)。

#### b. 重分类

**流程：**  打开System Toolboxes --> Spatial Analyst Tools.tbx --> Reclass --> Reclassify工具，需要填写或者修改的选项有`Input raster`、`Output raster`、还要打开`Classification`选项。

![image-20200826090728378](https://cdn.jsdelivr.net/gh/Rsweater/images/img/image-20200826090728378.png)

**注意：** 

1. 在`classification选项卡`中选过标准差，可能分类结果不是 正好就是5类，这个没关系，4-7类<sup>[1]</sup>都是可以的.
2. 可以统一下每个等级颜色。右击图层管理的色条就可以换。![image-20200826094056336](https://cdn.jsdelivr.net/gh/Rsweater/images/img/image-20200826094056336.png)

#### c. 计算城市热岛强度指数

**公式：** 

  $$H_i = \frac{T_i - T_{min}}{T_{max} - T_{min}} ^{[2]}    \tag{1}$$

式中：H<sub>i</sub>为第 i 个像元所对应的热场强度指数，Ti 为第 i 个像元所对应的地表温度，T<sub>min</sub>。为图像区域的有效最低地表温度，T<sub>max</sub>为图像区域对应的有效最高地表温度。热场强度指数值越大，表明处于热岛范围的可能性越大。

 简单说，就是对地表温度进行归一化处理。

**ArcGIS操作：**

​ 在 `ArcToolBox` 中，打开`Spatial Analyst-Overlay`（叠加分析）-Fuzzy Membership（模糊隶属度），System Toolboxes --> Spatial Analyst Tools.tbx --> Overlay --> Fuzzy Membership。

![image-20200826095500522](https://gitee.com/Rsweater_admin/Blog_Images/raw/master/img/image-20200826095500522.png)

​ 打开`Fuzzy Membership`之后，填写输入、输出路径，点选`Liner`类型![image-20200826095825062](https://gitee.com/Rsweater_admin/Blog_Images/raw/master/img/image-20200826095825062.png)

**参考文献：**

[1].[薛万蓉, 卢正, 但尚铭,等. 基于热红外遥感的四川省内江市城市热岛效应评价[J]. 测绘与空间地理信息, 2012(04):49-52.](https://xueshu.baidu.com/usercenter/paper/show?paperid=5eae86f6ba30b8230727059c364225f8&site=xueshu_se&hitarticle=1)

[2].[叶彩华, 刘勇洪, 刘伟东,等. 城市地表热环境遥感监测指标研究及应用[J]. 气象科技, 2011(01):95-101.](https://xueshu.baidu.com/usercenter/paper/show?paperid=fade569aad173a72f0dc9b98762489b5&site=xueshu_se)
