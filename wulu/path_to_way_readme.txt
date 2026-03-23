
无路网数据合并入库

输入：path_up2.shp 输出：pg数据库中ht_v_way表、ht_v_node表(注：运行前需确认ht_v_node表为原始数据表)

cd path_to_way_py python3 path_to_way.py dbname user password host post schema rootPath/ (其中rootPath/link文件夹包含步骤2生成的path_up2.shp文件以及整体无路数据范围road.sqlite), 生成最终的ht_v_way和ht_v_node表，用于valhalla编译。注：运行环境qgis版本需为3.28以上
代码详细流程
1、QGIS初始化
2、数据库连接
3、备份数据表
4、取高速与非高速部分存入中间表
5、创建无路中间表、更新道路表结构（增加无路属性）
6、打断无路数据
7、打断有路数据
8、打断后的无路文件入库
9、删除与高速相交无路数据
10、打断后的原始道路文件入库（追加至高速部分中间表）
11、merge_path：无路数据去重；更新id号（保留原始wayid（9-63位保存原始wayid，0-8位存被分割的序列号），无路部分从1开始，左移9位存储）；有路无路数据合并
12、根据level更新无路部分道路等级；更新无路部分名称
13、生成node表，更新way表nodeid
14、删除s_node、e_node相同的link（包含正反向）（优先删除无路link）
15、删除中间表，关闭数据库连接

