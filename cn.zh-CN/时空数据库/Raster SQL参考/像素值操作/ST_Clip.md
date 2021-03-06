# ST\_Clip {#reference_lpr_zgn_qfb .reference}

对raster对象进行裁剪操作。

## 语法 {#section_wkk_wcn_qfb .section}

``` {#codeblock_l94_qyw_bm5}
bytea ST_Clip(raster raster_obj,integer pyramidLevel,  box extent, BoxType boxType);
bytea ST_Clip(raster raster_obj,integer pyramidLevel,  box extent, BoxType boxType, integer destSrid);
record ST_Clip(raster raster_obj,
                     geometry geom,
                     integer pyramidLevel default 0,
                     cstring bands default '',  /* All bands */
                     float8[] nodata default NULL,
                     cstring clipOption default '',
                     cstring storageOption default '',
                     out box outwindow,
                     out bytea rasterblob)
```

## 参数 {#section_whn_ddn_qfb .section}

|参数名称|描述|
|:---|:-|
|raster\_obj|需要裁剪的raster对象。|
|pyramidLevel|金字塔层级。|
|extent|需要裁剪的范围，格式为'\(\(minX,minY\),\(maxX,maxY\)\)'。|
|boxType|范围的类型，只能是以下一种： -   Raster （像元坐标）
-   World （世界坐标）

 |
|destSrid|指定的输出像元子集的空间参考值。|
|geometry|需要裁剪的geometry对象。|
|bands|需要裁剪的波段，用'0-2'或者‘1,2,3’ 这种形式表示，以0开始。 默认为'', 表示裁剪所有的波段。|
|nodata|用float8\[\]表示的nodata数值。 如果数值个数少于波段数量，则使用波段设置的nodata值填充。如果波段未设置nodata，则用0填充。|
|clipOption|json字符串表示的裁剪选项。|
|storageOption|json字符串表示的返回结果的存储选项。|

clipOption参数如下。

|参数名称|类型|默认值|描述|
|:---|--|---|:-|
|window\_clip|bool|false|是否使用geometry的外包框进行裁剪。取值： -   true：使用geometry的MBR裁剪；
-   false：使用geometry对象裁剪。

 |

storageOption参数如下。

|参数名称|类型|默认值|描述|
|:---|--|---|:-|
|compression|string|lz4|压缩算法类型。取值： -   none
-   jpeg
-   zlib
-   png
-   lzo
-   lz4

 |
|quality|integer|75|压缩质量，只针对jpeg压缩算法。|
|interleaving|string|和原始raster一致|交错方式。取值： -   bip：Band interleaved by pixel；
-   bil：Band nterleaved by pixel；
-   bsq：Band Sequential。

 |
|endian|string|和原始raster一致|字节序。取值： -   NDR：Little endian；
-   XDR：Big endian。

 |

## 描述 {#section_f2z_ctn_qfb .section}

默认的裁剪缓存为100MB，代表最多只能裁剪出100MB大小的结果数据，如果需要调整返回结果大小，可使用参数 ganos.raster.clip\_max\_buffer\_size 设置缓存的大小。

## 示例 {#section_xhh_zfn_qfb .section}

``` {#codeblock_jye_3zz_pbq}
Select ST_Clip(raster_obj, 0, '((128.980,30.0),(129.0,30.2))', 'World');
Select ST_Clip(raster_obj, 0, '((128.980,30.0),(129.0,30.2))', 'World'， 4326);

--  使用geometry裁剪
-- 都是用默认值裁剪
SELECT (ST_CLIP(rast, ST_geomfromtext('Polygon((0 0, 45 45, 90 45, 45 0, 0 0))', 4326), 0)).* from clip_table where id =1

-- 使用白色作为背景色填充并且压缩为png图片
SELECT (ST_CLIP(rast, ST_geomfromtext('Polygon((0 0, 45 45, 90 45, 45 0, 0 0))', 4326), 0, '', ARRAY[254,254,254], '', '{"compression":"png","interleaving":"bip"}')).* from clip_table where id =1;

-- 使用geometry的窗口裁剪
SELECT (ST_CLIP(rast, ST_geomfromtext('Polygon((0 0, 45 45, 90 45, 45 0, 0 0))', 4326), 0, '', NULL, '{"window_clip":true}', '')).* from clip_table where id =1;
```

