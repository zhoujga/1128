# GDAL OGR SMS Driver

SMS (Standard Mapping Service) 电子海图/地形图格式的 GDAL OGR 驱动。

## 文件说明

- `ogrsms.so` - GDAL OGR SMS 驱动插件
- `ogr2ogr_sms.py` - 命令行转换工具
- `ogrinfo_sms` - 命令行信息查看工具

## 安装

### 方式一：直接使用 Python 工具（推荐）

将 `ogrsms.so` 和 `ogr2ogr_sms.py` 复制到同一目录即可使用。

### 方式二：编译到 GDAL 源码

#### 1. 复制驱动源码

```bash
cp -r /mnt/bigdata/work/data_compile/gdal_source/ogr/ogrsf_frmts/sms /path/to/gdal_source/ogr/ogrsf_frmts/
```

#### 2. 修改 CMakeLists.txt

在 `/path/to/gdal_source/ogr/ogrsf_frmts/CMakeLists.txt` 末尾添加：

```cmake
# SMS (Standard Mapping Service) driver
ogr_optional_driver(sms SMS)
```

#### 3. 使用 GDAL Docker 镜像编译

```bash
# 创建构建目录
mkdir -p /path/to/gdal_build
cd /path/to/gdal_build

# 运行 Docker 镜像编译
docker run --rm \
    -v /path/to/gdal_source:/source \
    -v /path/to/gdal_build:/build \
    osgeo/gdal:ubuntu-full-3.4.1 \
    sh -c "cd /build && \
           cmake /source \
               -DCMAKE_BUILD_TYPE=Release \
               -DOGR_ENABLE_DRIVER_SMS=ON \
               -DBUILD_TESTING=OFF \
               -DBUILD_PYTHON_BINDINGS=OFF && \
           make -j$(nproc) && \
           make install"
```

#### 4. 使用编译好的 ogr2ogr

```bash
# 设置环境变量
export PATH=/usr/local/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

# 查看 SMS 驱动
ogrinfo --formats | grep SMS

# 转换文件
ogr2ogr -f GPKG output.gpkg input.SMS
ogr2ogr -f GeoJSON output.geojson input.SMS

# 查看信息
ogrinfo input.SMS
```

## 使用方法

### 1. 查看 SMS 文件信息

```bash
./ogr2ogr_sms.py --info test_data/5w-test-sms_utf8/DN08511111/DN08511111.SMS
```

输出示例:
```
Driver: SMS (Standard Mapping Service)
File: test_data/5w-test-sms_utf8/DN08511111/DN08511111.SMS
Layer count: 11
Projection: PROJCS["CGCS_2000_117"...

Layers:
------------------------------------------------------------
  0: 测量控制点                -      0 features
  1: 工内农社会文化              -     48 features
  2: 居民地                  -   3279 features
  ...
------------------------------------------------------------
Total features: 12604
```

### 2. 转换为 GeoPackage (推荐)

```bash
./ogr2ogr_sms.py -f GPKG -o output.gpkg input.SMS
```

### 3. 转换为 GeoJSON

```bash
./ogr2ogr_sms.py -f GeoJSON -o output.geojson input.SMS
```

### 4. 选择特定图层转换

```bash
./ogr2ogr_sms.py -f GPKG -o output.gpkg input.SMS -l "居民地"
```

### 5. 转换所有图层到 GPKG

```bash
./ogr2ogr_sms.py -f GPKG -o all_layers.gpkg test_data/5w-test-sms_utf8/DN08511111/DN08511111.SMS
```

## 命令行选项

| 选项 | 说明 |
|------|------|
| `--info` | 显示文件信息 |
| `-f format` | 输出格式 (默认: GeoJSON) |
| `-o output` | 输出文件路径 |
| `-l layername` | 选择特定图层 |

## 支持的输出格式

| 格式 | 说明 |
|------|------|
| **GPKG** | GeoPackage - 推荐，支持多图层和中文字段 |
| **GeoJSON** | 支持单图层 |
| **ESRI Shapefile** | 不推荐，不支持中文字段名 |

## 图层类型

| 代码 | 图层名称 |
|------|----------|
| A | 测量控制点 |
| B | 工农业社会文化 |
| C | 居民地 |
| D | 陆地交通运输 |
| E | 管线与桓栅 |
| F | 水域陆地 |
| G | 水深及底质 |
| H | 礁石沉船障碍物 |
| I | 水文 |
| J | 陆地地貌及土质 |
| K | 境界与政区 |
| L | 植被 |
| M | 地磁要素 |
| N | 助航设备及航道 |
| O | 海上区域界限 |
| P | 航空要素 |
| Q | 军事区域 |
| R | 注记 |
| T | 图外信息 |
| W | 军事打击目标 |

## Python API 使用

```python
from osgeo import gdal
import ctypes

# 加载 SMS 驱动
lib = ctypes.CDLL('ogrsms.so')
lib._Z14RegisterOGRSMSv()

# 打开数据集
ds = gdal.OpenEx('input.SMS', gdal.OF_VECTOR | gdal.OF_READONLY)

# 遍历图层
for i in range(ds.GetLayerCount()):
    layer = ds.GetLayer(i)
    print(f'{layer.GetName()}: {layer.GetFeatureCount()} features')

# 获取特定图层
layer = ds.GetLayerByName('居民地')
```

## 注意事项

1. 坐标转换根据地图比例尺进行 (1:10000 到 1:1000000)
2. Shapefile 不支持中文和复杂几何，建议使用 GPKG
3. 部分图层可能为空 (0 features)
4. 警告 "geometry type inserted into layer of geometry type POINT" 是正常的，不影响数据

## 测试数据

测试数据位于 `test_data/5w-test-sms_utf8/` 目录。
