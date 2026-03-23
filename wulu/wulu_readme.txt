
全地形数据编译
1、SMS批量转换

输入:地形图J标数据

输出：shp。按图幅划分。工具：SMS批量导出工具
2、按数据生产区域合并SHP（5类)
2.1 制作数据生产区域SHP。

如果区域大生产时候会特别慢，需要将区域多划分几个矢量。这一步需要在QGIS里面，手动绘制矢量区域。绘制的矢量需要存储为spatialite数据库文件。文件名称road.sqlite。 表名outline。如果要划分多个区域，需要生成多份road.sqlite。
2.2 要在QGIS中执行py脚本，NO_ROAD(merge_shp.py)这个文件中拷贝代码到QGIS中执行，需要替换shp路径和输出的路径。合并的数据包括五类（工农业社会文化设施、居民地与附属设施、陆地地貌及土质、水域陆地、植被）
2.3 在QGIS中使用2.1生产的SHP,对合并后的全部数据进行空间拓扑，查出相交的部分，另存为部分区域数据。比如2.1划分了4个区域，2.2生产出来的5类数据就需要分别与这4个、按数据生产区域合并SHP区域求交，最终生产出4个区域的5类数据。
3 将5类SHP数据转成栅格TIF

在QGIS 上使用Rasterize(vector to raster)
1 先把矢量加载到QGIS中，转成3857坐标系。再进行栅格化。

1 工农业社会文化设施 1.1 第二个参数，字段空着。 1.2 A fixed value to burn 选 1 1.3 Output raster size units 选择 georeferenced units 1.4 Width 10.0 1.5 Height 10.0 1.6 Output extent 从右侧选择该图层范围 1.7 Notdata 默认值0 1.8 Rasterized 中选择 save to file 保存成tif,名称为固定名称facility_3857.tif.
2 其他4类数据。

2.1 第二个参数选地形图编码属性字段。有些数据可能不叫编码得从协议里面看字段名。 2.2 默认值选0 2.3 后续的参数都一样。 2.4 导出名称：植被：vegetation_3857.tif,水域陆地:river_system_3857.tif,陆地地貌:soil_3857.tif,居民地:residential_area_3857.tif,工农业社会:facility_3857.tif 2.5 栅格投影坐标另存4326坐标系，输出名称固定 facility.tif,residential_area.tif,river_system.tif,soil.tif,vegetation.tif
4 Dem 数据转坡度tif数据 坐标系4326

QGIS 中工具箱 slop Z factor 0.00001 输出名称 slope.tif
5 生成H3格网

no_road 路径 36 行 parentCelllevel(constants.h)

road.sqlite 范围数据 ./build/executor /app/data/ cells raw_tif 目录里面都放road.sqlite

Docker run -it -v $(pwd):/app wulu:v2.0 /bin/bash cd /app cd no_road/ ./build/executor /app/data/
6 军标数据入库

1.0 pgrouting 创建数据库 2 创建数据库 jb_db 3 创建postgis扩展, create extension postgis 4 2.compile_data 数据入库 config.json 5 军标文本放到2.compile_data/jb_txt Docker-compose up jbtxt2jbdb
7 noroad 军交数据和无路网数据融合

noroad 文件夹下 path_to_way_py Docker run -it -v $(pwd):/app wulu:v2.0 /bin/bash Python3 path_to_way_py

第一步代码先执行1-5 到220行，第一步执行，后面注释掉。有ht_v_way 第二步QGIS工具Split with line 第一次 path 为输入图层

Save to GeoPackage path_temp1.gpkg output 不用管

第二次 path 为split图层 Ht_v_way_tmp为输入图层 Save to GeoPackage path_temp1 output 不用管生成数据放到 noroad/noroad/path_to_way_py 下跑8-15步，代码240行
8 运行编译规划

schema docker-compose up route config.json 中数据库参数第8行，schema名称对应到root:[schema:] 生成数据在output/offline 目录下navi_data

sudo docker-compose up routecompile
9 sudo docker-compose up backcompile

20 行代码 Back_compile.sh
10 linkinfo.db 执行 linkinfo.py

修改schema

