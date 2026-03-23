
1、先编译h3 生成 对应的dll 复制到lib文件夹中
2、h3编译 cmake .. -DBUILD_TESTING=OFF -DBUILD_SHARED_LIBS=ON
3、windows上依赖库 boost sqlite gdal 使用OSGeo4W中的库，在cmake中指定CMAKE_PREFIX_PATH

cmake -G "Visual Studio 15 2017 Win64" .. D:\htht\A\A\pack\release-1928-x64-gdal-3-7-2-mapserver-8-0-1\bin D:/htht/A/A/pack/release-1928-x64-gdal-3-7-2-mapserver-8-0-1-libs

编译流程分为 1.contours-cell。2.locate_center_point。3.build_link。4.build_level。 四个步骤目前所有的配置信息都整合到 inc/Constants.h 的文件下，替换 const std::string root = "F:\share\data2\"; 目录为当前需要执行的目录的根目录。文件夹结构如下 root-- |--raw_tif (用于存放所有的原始tif 文件信息) |--center_points(代表点 shape 文件) |--link (路网shape path.shp) |--cells-- |--level11(11级别路网shape文件) |--level13(11级别路网shape文件) |-road.sqlite

按照上面的文件存储方式，需要替换 raw tif 里面对应的文件信息，目前程序读取的 tif 文件名称命名如下： building,green_diameter,green_type,landform_high,landform_type,slope_ha,ware,water. 同时需要替换掉 cells 目录下面的 road.sqlite 为对应的裁切范围。

注意事项： 1.build_level 最后一步之前需要合并所有的build_link步骤生成的 path.shp 文件再执行抽层的操作。 2.build_link 最后生成link的步骤已经添加了去除冗余形状点的处理。 3.这个版本没有添加超过 8 个link的 校验，这个校验程序是一个 名叫 post_processor python 脚本，在根目录下下。 4.目前选取中心点和build_link 用于判断的共同参数都写在了 Constants.h common params 里面。 5.build_link 建立拓扑关系中用于计算路径weight的方法叫做 getLinkCost，目前只是对于不同的 路况如说水系高差前面乘上了一个自定义的系数。 6.build_link 431 行对于最终这条link的 slope 属性设置是计算的平均值，其他的 除了type 之外都是计算的和。
全地形数据制作流程

输入：.tif文件 road.sqlite 输出：path_up2.shp

制作流程： 1、 准备文件夹，目录结构如下， root-- |--raw_tif (用于存放所有的原始tif 文件信息) |--center_points(代表点 shape 文件) |--link (路网shape) |--cells-- |--level11(11级别路网shape文件) |--level13(11级别路网shape文件) |-road.sqlite(其中图层名为“outline”) 按照上面的文件存储方式，需要替换 raw tif 里面对应的文件信息，目前程序读取的 tif 文件名称命名如下： vegetation,river_system,slope,residential_area,facility,soil. 同时需要替换掉 cells 目录下面的 road.sqlite 为对应的无路数据范围(其中图层名为“outline”)。

2、 编译no_road(cd build; cmake ..;make -j8)，运行 ./build/executor rootPath/ ，在link文件夹下生成无路路网文件path_up2.shp。注：若输入为多块数据，可准备多个相同结构文件夹同时执行（也可以多次执行，保留path.shp文件），多块数据全部执行完成后，将各link文件夹下的path.shp合并（可以使用qgis操作），存放至link文件夹下(文件名为path.shp)，再执行./build/build_level rootPath/ ,生成path_up2.shp。
