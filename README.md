# Cesium 学习记录

## 一、本地地图 / 图层服务 / 在线地图 / 地形数据
### 1. 本地地图： 地图瓦片资源下载到本地文件夹，通过nginx代理访问文件
```js
// 加载本地地图示例
viewer = new Cesium.Viewer(element, {
  imageryProvider: new Cesium.UrlTemplateImageryProvider({
    url: 'http://192.168.1.2:8080/map/{z}/{x}/{y}.jpg'
  }),
})
```
### 2. 图层服务： 有三种新式  wms wmts tms
在 Cesium 中，常用的三种地图服务协议是 WMS、WMTS 和 TMS，它们都有自己的特点和使用场景。以下是它们的区别：

#### 2.1 **WMS（Web Map Service）**
   - **简介：** WMS 是由 OGC（Open Geospatial Consortium）定义的标准，它是一种基于请求-响应模式的服务，用于请求地图图像。WMS 请求包含诸如地图的边界框、层次、图像格式等参数，服务器根据这些参数动态生成图像并返回给客户端。
   - **特点：**
     - **动态生成：** 每次请求都是动态生成的图像，可以包含最新的数据（例如实时天气）。
     - **按需请求：** 可以请求任意范围的图像，不受预定义瓦片的限制。
     - **支持透明度、样式和时间维度：** 可以请求具有透明背景的图像、调整图层样式，甚至是基于时间的动画。
   - **适用场景：**
     - 需要高度定制化的地图显示。
     - 地图数据频繁更新，或者需要实时数据。
   
   - **缺点：**
     - **性能较低：** 因为图像是动态生成的，处理时间可能较长，特别是当请求的范围较大时。

#### 2.2 **WMTS（Web Map Tile Service）**
   - **简介：** WMTS 是 WMS 的一种扩展，它也是由 OGC 定义的标准。与 WMS 不同的是，WMTS 提供预生成的瓦片图像，这些瓦片通常以固定的缩放级别和分辨率存储在服务器上。客户端请求时只需获取对应的瓦片。
   - **特点：**
     - **高性能：** 由于瓦片是预生成的，WMTS 可以快速响应客户端请求。
     - **固定缩放级别：** 只能请求预定义的缩放级别的瓦片，灵活性较低。
     - **标准化：** 瓦片的结构是标准化的，使得缓存和复用更加容易。
   - **适用场景：**
     - 需要高性能、快速响应的地图应用。
     - 地图数据相对稳定，不需要频繁更新。
     - 支持固定的缩放级别和分辨率。

   - **缺点：**
     - **灵活性较低：** 只能请求固定的缩放级别和瓦片大小，无法动态调整。

#### 2.3 **TMS（Tile Map Service）**
   - **简介：** TMS 是一种非标准化但广泛使用的瓦片服务协议。它定义了一个简单的基于 URL 的模式来请求瓦片，瓦片的命名和组织方式相对简单。TMS 瓦片的排列从左下角（0,0）开始，而其他服务如 WMTS 通常从左上角开始。
   - **特点：**
     - **简单性：** URL 结构简单，容易理解和实现。
     - **自定义性：** 可以根据需要自定义瓦片的结构和请求方式。
     - **适用于自定义瓦片服务：** 如果你有自己的瓦片服务器，TMS 是一种简单易用的方案。
   - **适用场景：**
     - 自定义的瓦片服务。
     - 需要简单、轻量级的瓦片请求协议。
     - 非标准化场景下的地图服务实现。

   - **缺点：**
     - **非标准化：** 没有严格的标准，可能导致兼容性问题。
     - **通常没有元数据：** 缺乏服务能力描述和元数据支持。

#### **总结**
- **WMS** 适用于需要动态、定制化和最新数据的场景，但性能相对较低。
- **WMTS** 适用于需要高性能、快速响应的场景，数据通常是静态的或相对稳定的。
- **TMS** 简单易用，适合自定义瓦片服务，但缺乏标准化的规范。
```js
// 我们用 WMTS 来示例
let wmtsLayer = new Cesium.WebMapTileServiceImageryProvider({
    url: 'https://example.com/wmts',  // WMTS 服务的基本 URL
    layer: 'your_layer_name',         // 图层名称
    style: 'default',                 // 样式名称
    format: 'image/png',              // 瓦片格式，例如 'image/png' 或 'image/jpeg'
    tileMatrixSetID: 'EPSG:3857',     // 瓦片矩阵集，通常是 EPSG:3857 或 EPSG:4326
    maximumLevel: 19,                 // 最大缩放级别
    tileMatrixLabels: [
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '10', '11', '12', '13', '14', '15', '16', '17', '18', '19'
    ],// 每个缩放级别对应的标签，一般使用数字或级别名称。
    tilingScheme: new Cesium.WebMercatorTilingScheme(), // 根据坐标系选择合适的 tilingScheme
    tileWidth: 256,                  // 瓦片宽度
    tileHeight: 256,                 // 瓦片高度
    credit: new Cesium.Credit('WMTS Layer Example') // 提供数据来源的信用信息
});

```
### 3. 线上图层：
例如： 百度 谷歌等一些大厂提供的线上地图
```text
设置百度地图的 URL 模板：

使用 UrlTemplateImageryProvider 加载百度地图的瓦片服务。百度地图的瓦片服务 URL 模板为 https://online{switch:a,b,c}.map.bdimg.com/onlinelabel/?qt=tile&x={x}&y={y}&z={z}&styles=sl&scaler=1&p=1。
自定义标签 customTags：百度地图的瓦片系统与标准 Web Mercator 不同，因此需要调整 x 和 y 坐标。代码中使用 customTags 来计算正确的瓦片坐标。
```
```js
let baiduImageryProvider = new Cesium.UrlTemplateImageryProvider({
    url : 'https://online{switch:a,b,c}.map.bdimg.com/onlinelabel/?qt=tile&x={x}&y={y}&z={z}&styles=sl&scaler=1&p=1',
    tilingScheme : new Cesium.WebMercatorTilingScheme(),
    maximumLevel: 18,
    customTags: {
        x: function(imageryProvider, x, y, level) {
            return x - Math.pow(2, level - 1);
        },
        y: function(imageryProvider, x, y, level) {
            return Math.pow(2, level - 1) - 1 - y;
        }
    }
});
```
### 4.地形数据
在 Cesium 中，地形数据加载是通过 Cesium.TerrainProvider 类及其子类来实现的。Cesium 支持多种地形数据格式，如 Cesium's own quantized-mesh, heightmap, 和其他标准的地形数据格式。
```js
let viewer = new Cesium.Viewer('cesiumContainer', {
    terrainProvider: Cesium.createWorldTerrain(), // 使用 Cesium World Terrain 这是cesium官方提供
    baseLayerPicker: false // 禁用图层选择器（可选）
});

// 自定义地形数据
let customTerrainProvider = new Cesium.CesiumTerrainProvider({
    url : 'https://your.custom.terrain.service/tilesets/terrain',
    requestVertexNormals : true, // 请求法向量数据，用于更好的光照效果
    requestWaterMask : true      // 请求水面掩码，用于水面效果
});
viewer.terrainProvider = customTerrainProvider;
```
### 5.切换图层
```js
// 定义多个 ImageryProvider
let imageryProvider1 = new Cesium.UrlTemplateImageryProvider({ url: 'https://xxx/{z}/{x}/{y}.png'});
let imageryProvider2 = new Cesium.UrlTemplateImageryProvider({ url: 'https://xxx/{z}/{x}/{y}.png'});
// 当前的图层变量，用于存储当前显示的图层
let currentLayer;
// 切换图层
// 函数：添加图层并删除之前的图层
function addLayer(imageryProvider) {
    // 确保 imageryProvider 已经加载完毕
    imageryProvider.readyPromise.then(function() {
        // 删除当前显示的图层（如果存在）
        if (currentLayer) {
            viewer.imageryLayers.remove(currentLayer);
        }
        // 添加新的图层
        currentLayer = viewer.imageryLayers.addImageryProvider(imageryProvider);
        console.log('Layer added:', imageryProvider);
    }).otherwise(function(error) {
        console.error('An error occurred while loading the imagery provider:', error);
    });
}
//示例：加载第一个图层
addLayer(imageryProvider1);
// addLayer(imageryProvider2);
```



## 二、标绘
在这之前得学习一下 抽象的概念
往地图中添加实体之前最好给他归类，相同类型的放到一个集合中方便后面控制显影。

在许多图形库和 GIS 系统中，**GraphicLayer 图形层**（或类似概念）用于组织和管理多个图形对象。它允许将图形对象分组到一个层中，以便于管理、渲染和控制可见性等。
cesium 中虽然没有直接名为 "Layer" 的概念，但有多个类和机制能够实现类似于 "图层" 的功能。
- **Imagery Layer（影像图层）**
  - 类： **Cesium.ImageryLayer** 
  - 用于管理和显示影像图层，如卫星影像和地图瓦片 ImageryLayer 是在地球表面上显示影像数据的主要方式，通常叠加在地形图上。
  - 管理类： **Cesium.ImageryLayerCollection** 是一个用于管理所有影像图层的集合。
  ```js
   添加图层：viewer.imageryLayers.addImageryProvider()
   控制可见性：layer.show = true/false
   调整顺序：imageryLayers.raise(layer) 或 imageryLayers.lower(layer)

  let viewer = new Cesium.Viewer('cesiumContainer');
  let imageryLayers = viewer.imageryLayers;

  // 添加一个影像图层
  let layer = imageryLayers.addImageryProvider(new Cesium.ArcGisMapServerImageryProvider({
      url : 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer'
  }));

  // 控制图层的可见性
  layer.show = false; // 隐藏该图层
  ```
  

- **Terrain Layer（地形图层）**
  - 类： **Cesium.CesiumTerrainProvider** 
  - 用于管理和显示地形数据的图层。地形图层通常与影像图层结合使用，以增强地理可视化效果。
  ```text
    设置地形提供程序：viewer.terrainProvider = new Cesium.CesiumTerrainProvider(...)
    控制地形图层的可见性：通过设置 viewer.terrainProvider 为 Cesium.EllipsoidTerrainProvider() 可以禁用地形。
  ```
  ```js
  let viewer = new Cesium.Viewer('cesiumContainer');
  // 添加地形图层
  viewer.terrainProvider = new Cesium.CesiumTerrainProvider({
      url: 'https://assets.agi.com/stk-terrain/world'
  });

  ```
- **DataSource（数据源）** 
  - 类： **Cesium.DataSource**
  - 用于加载和管理复杂的地理数据，如 GeoJSON、KML、CZML 文件。每个数据源可以看作一个图层，里面包含多个实体（Entities）。
  - 管理类：**Cesium.DataSourceCollection** 是一个用于管理数据源的集合，viewer.dataSources 是 DataSourceCollection 的实例。
  ```js
  // 相关操作
  添加数据源：viewer.dataSources.add(dataSource)
  控制可见性：dataSource.show = true/false

  let viewer = new Cesium.Viewer('cesiumContainer');

  // 加载一个 GeoJSON 数据源
  Cesium.GeoJsonDataSource.load('path/to/data.geojson').then(function(dataSource) {
      viewer.dataSources.add(dataSource);
      // 控制数据源的可见性
      dataSource.show = false; // 隐藏该数据源
  });

  ```
- **Primitive Collection（原始图元集合）**
  - 类： **Cesium.PrimitiveCollection**
  - 描述：管理和组织自定义几何体或图形对象。虽然不是直接的“图层”，但可以用作自定义图形对象的容器，类似于图层的功能。
  ```js
  // 相关操作
  添加到场景：viewer.scene.primitives.add(primitiveCollection)
  控制可见性：primitiveCollection.show = true/false

  let viewer = new Cesium.Viewer('cesiumContainer');

  // 创建 PrimitiveCollection
  let primitiveCollection = new Cesium.PrimitiveCollection();

  // 添加自定义几何体
  primitiveCollection.add(new Cesium.Primitive({
      geometryInstances : new Cesium.GeometryInstance({
          geometry : new Cesium.BoxGeometry({
              vertexFormat : Cesium.PerInstanceColorAppearance.VERTEX_FORMAT,
              maximum : new Cesium.Cartesian3(1.0, 1.0, 1.0),
              minimum : new Cesium.Cartesian3(-1.0, -1.0, -1.0)
          }),
          modelMatrix : Cesium.Transforms.eastNorthUpToFixedFrame(Cesium.Cartesian3.fromDegrees(-75.59777, 40.03883)),
          attributes : {
              color : Cesium.ColorGeometryInstanceAttribute.fromColor(Cesium.Color.RED)
          }
      }),
      appearance : new Cesium.PerInstanceColorAppearance()
  }));

  // 将集合添加到场景
  viewer.scene.primitives.add(primitiveCollection);
  ```


### 1. 点
在地图中标点，画出点的类型
 - 类： PointGraphics
 - 描述 位于包含 Entity 的位置的图形点。
 - [中文官方文档](http://www.cesium.xin/cesium/cn/Documentation1.62/PointGraphics.html)
```js
//示例
let viewer = new Cesium.Viewer('cesiumContainer');
// 添加一个实体，并使用 PointGraphics 绘制一个点
let pointEntity = viewer.entities.add({
    position : Cesium.Cartesian3.fromDegrees(-75.59777, 40.03883),
    point : {
        // 设置点的颜色
        color : Cesium.Color.RED,
        // 设置点的大小（像素）
        pixelSize : 10,
        // 设置点的透明度
        translucencyByDistance : new Cesium.NearFarScalar(1.5e2, 1.0, 1.5e7, 0.5),
        // 设置点的轮廓颜色
        outlineColor : Cesium.Color.YELLOW,
        // 设置点的轮廓宽度（像素）
        outlineWidth : 2,
        // 是否显示点
        show : true,
        // 指定点的高度参考
        heightReference : Cesium.HeightReference.CLAMP_TO_GROUND,
        // 点与摄像机距离的缩放因子
        distanceDisplayCondition : new Cesium.DistanceDisplayCondition(0.0, 7.0e6),
        // 指定点的垂直偏移（米）
        verticalOrigin : Cesium.VerticalOrigin.BOTTOM,
        // 是否使用深度检测，默认为 true
        disableDepthTestDistance : Number.POSITIVE_INFINITY
    }
});

```

### 2. 线
 - 类： PolylineGraphics
 - 描述折线。前两个位置定义线段，并且每个其他位置都从前一个位置定义了一个线段。细分市场可以是线性连接点，大弧或固定在地形上。
 - [中文官方文档](http://www.cesium.xin/cesium/cn/Documentation1.62/PolylineGraphics.html?classFilter=po)
  - 主要线条类型说明
    - 实线: 普通的连续线条。
    - 虚线: 通过 PolylineDashMaterialProperty 设置的虚线。
    - 边框线: 通过 PolylineOutlineMaterialProperty 设置的带有外边框的线条。
    - 发光线: 通过 PolylineGlowMaterialProperty 设置的具有发光效果的线条。
    - 箭头线: 通过 PolylineArrowMaterialProperty 设置的带箭头的线条。
    - 动态线: 通过 CallbackProperty 实现动态颜色变化的线条。
    - 纹理线: 通过 ImageMaterialProperty 使用自定义图像纹理绘制的线条。
    - 弧线: 使用自定义函数 createArcPositions 生成弧形路径的线条。
    - 贴地线: 设置 clampToGround 为 true，使线条贴合地表。
    - 高度设置: 通过 fromDegreesArrayHeights 设置线条的高度。

```js
//示例
// 创建 Cesium Viewer
let viewer = new Cesium.Viewer('cesiumContainer');
// 实线
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 35, -125, 35]),
        width : 5,
        material : Cesium.Color.BLUE,
        clampToGround : false, // 设置是否贴地
    }
});

// 虚线
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 33, -125, 33]),
        width : 5,
        material : new Cesium.PolylineDashMaterialProperty({
            color: Cesium.Color.RED,
            dashLength: 16.0
        }),
        clampToGround : false,
    }
});

// 边框线
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 31, -125, 31]),
        width : 5,
        material : new Cesium.PolylineOutlineMaterialProperty({
            color: Cesium.Color.YELLOW,
            outlineWidth: 2,
            outlineColor: Cesium.Color.BLACK,
        }),
        clampToGround : false,
    }
});

// 发光线
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 29, -125, 29]),
        width : 10,
        material : new Cesium.PolylineGlowMaterialProperty({
            color: Cesium.Color.ORANGE,
            glowPower: 0.2,
        }),
        clampToGround : false,
    }
});

// 箭头线
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 27, -125, 27]),
        width : 5,
        material : new Cesium.PolylineArrowMaterialProperty(Cesium.Color.GREEN),
        clampToGround : false,
    }
});

// 动态线（动态颜色变化）
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 25, -125, 25]),
        width : 5,
        material : new Cesium.ColorMaterialProperty(new Cesium.CallbackProperty(function() {
            return Cesium.Color.fromRandom();
        }, false)),
        clampToGround : false,
    }
});

// 纹理线
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 23, -125, 23]),
        width : 5,
        material : new Cesium.ImageMaterialProperty({
            image: 'path/to/your/texture.png', // 这里替换为实际的纹理图像路径
            repeat: new Cesium.Cartesian2(10, 1)
        }),
        clampToGround : false,
    }
});

// 弧线
function createArcPositions(startLon, startLat, endLon, endLat) {
    let positions = [];
    let middlePoint = Cesium.Cartesian3.midpoint(
        Cesium.Cartesian3.fromDegrees(startLon, startLat),
        Cesium.Cartesian3.fromDegrees(endLon, endLat),
        new Cesium.Cartesian3()
    );

    let height = Cesium.Cartesian3.distance(
        Cesium.Cartesian3.fromDegrees(startLon, startLat),
        Cesium.Cartesian3.fromDegrees(endLon, endLat)
    ) / 10;

    middlePoint = Cesium.Cartesian3.multiplyByScalar(
        Cesium.Cartesian3.normalize(middlePoint, new Cesium.Cartesian3()),
        height,
        middlePoint
    );

    positions.push(Cesium.Cartesian3.fromDegrees(startLon, startLat));
    positions.push(middlePoint);
    positions.push(Cesium.Cartesian3.fromDegrees(endLon, endLat));
    return positions;
}

viewer.entities.add({
    polyline : {
        positions : createArcPositions(-75, 21, -125, 21),
        width : 5,
        material : Cesium.Color.CYAN,
        clampToGround : false,
    }
});

// 贴地线
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArray([-75, 19, -125, 19]),
        width : 5,
        material : Cesium.Color.MAGENTA,
        clampToGround : true, // 贴地
    }
});

// 高度设置
viewer.entities.add({
    polyline : {
        positions : Cesium.Cartesian3.fromDegreesArrayHeights([
            -75, 17, 100000, // 起点和高度（米）
            -125, 17, 100000 // 终点和高度（米）
        ]),
        width : 5,
        material : Cesium.Color.YELLOW,
        clampToGround : false,
    }
});
```

### 3. 图标
  - 类： BillboardGraphics
  - 描述位于包含 Entity 的位置的二维图标。
  - [中文官方文档](http://www.cesium.xin/cesium/cn/Documentation1.62/BillboardGraphics.html?classFilter=bi)
  - BillboardGraphics 的主要属性
    - image: 设置标注的图像，通常是图标或图片的 URL。
    - position: 设置标注的位置，可以使用 Cesium.Cartesian3.fromDegrees 设置经纬度。
    - scale: 设置图标的缩放比例。
    - rotation: 设置图标的旋转角度（以弧度为单位）。
    - color: 设置图标的颜色，通常用于调整图标的透明度或颜色变化。
    - horizontalOrigin 和 verticalOrigin: 设置图标的对齐方式。
    - heightReference: 设置图标的高度参考（例如 CLAMP_TO_GROUND 或 RELATIVE_TO_GROUND）。
    - alignedAxis: 设置图标对齐的轴方向，用于控制图标的朝向。
    - pixelOffset: 设置图标的像素偏移，用于微调图标在屏幕上的位置。
```js
// 创建 Cesium Viewer
let viewer = new Cesium.Viewer('cesiumContainer');

// 创建一个带图标的 Billboard
viewer.entities.add({
    position : Cesium.Cartesian3.fromDegrees(-75.59777, 40.03883),
    billboard : new Cesium.BillboardGraphics({
        image : 'path/to/icon.png', // 图标路径
        scale : 1.5, // 图标缩放比例
        rotation : Cesium.Math.toRadians(45), // 图标旋转角度
        color : Cesium.Color.WHITE.withAlpha(0.8), // 图标颜色与透明度
        heightReference : Cesium.HeightReference.CLAMP_TO_GROUND, // 贴地
    })
});

// 创建一个带文字的 Billboard
viewer.entities.add({
    position : Cesium.Cartesian3.fromDegrees(-75.59777, 40.03883),
    label : {
        text : 'Hello Cesium!',
        font : '24px Helvetica',
        fillColor : Cesium.Color.YELLOW,
        outlineColor : Cesium.Color.BLACK,
        outlineWidth : 2,
        style : Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin : Cesium.VerticalOrigin.BOTTOM, // 文字下对齐
        pixelOffset : new Cesium.Cartesian2(0, -25) // 像素偏移，避免与图标重叠
    },
    billboard : {
        image : new Cesium.PinBuilder().fromColor(Cesium.Color.ROYALBLUE, 48).toDataURL(), // 蓝色图钉图标
        verticalOrigin : Cesium.VerticalOrigin.BOTTOM,
        heightReference : Cesium.HeightReference.RELATIVE_TO_GROUND, // 相对于地形
    }
});

// 创建一个带旋转效果的 Billboard
viewer.entities.add({
    position : Cesium.Cartesian3.fromDegrees(-80.50, 35.14),
    billboard : new Cesium.BillboardGraphics({
        image : 'path/to/another-icon.png', // 另一图标路径
        scale : 1.0,
        alignedAxis : Cesium.Cartesian3.UNIT_Z, // 对齐Z轴
        rotation : new Cesium.CallbackProperty(function(time, result) {
            return Cesium.Math.toRadians((time.secondsOfDay % 360)); // 根据时间旋转
        }, false),
        heightReference : Cesium.HeightReference.CLAMP_TO_GROUND,
    })
});

// 使摄像机飞到标注所在的位置
viewer.zoomTo(viewer.entities);

```

### 4. 文字
  - 类： LabelGraphics
  - 描述位于包含位置的二维标签 Entity。
  - [中文官方文档](http://www.cesium.xin/cesium/cn/Documentation1.62/LabelGraphics.html?classFilter=LabelGraphics)
  - LabelGraphics 的主要属性
    - text: 标签的文本内容。
    - font: 标签的字体样式和大小（例如 '24px Helvetica'）。
    - fillColor: 标签文字的填充颜色。
    - outlineColor: 标签文字的轮廓颜色。
    - outlineWidth: 标签文字的轮廓宽度。
    - style: 标签的样式，包括填充、轮廓或者两者同时（Cesium.LabelStyle.FILL，Cesium.LabelStyle.OUTLINE，Cesium.LabelStyle.FILL_AND_OUTLINE）。
    - verticalOrigin: 标签的垂直对齐方式（Cesium.VerticalOrigin.BOTTOM，Cesium.VerticalOrigin.CENTER，Cesium.VerticalOrigin.TOP）。
    - horizontalOrigin: 标签的水平对齐方式（Cesium.HorizontalOrigin.LEFT，Cesium.HorizontalOrigin.CENTER，Cesium.HorizontalOrigin.RIGHT）。
    - pixelOffset: 标签的像素偏移，用于微调标签在屏幕上的位置。
    - scale: 标签的缩放比例。
    - translucencyByDistance: 通过距离来控制标签的透明度。
    - distanceDisplayCondition: 设置标签显示的距离范围。
    - heightReference: 标签的高度参考（例如 Cesium.HeightReference.CLAMP_TO_GROUND）。
    - showBackground: 是否显示背景。
    - backgroundColor: 背景颜色。
    - backgroundPadding: 背景的填充大小，用于调整背景与文字之间的空隙。
```js
// 创建 Cesium Viewer
let viewer = new Cesium.Viewer('cesiumContainer');

// 创建一个简单的文本标签
viewer.entities.add({
    position : Cesium.Cartesian3.fromDegrees(-75.1641667, 39.9522222), // 费城经纬度
    label : new Cesium.LabelGraphics({
        text : 'Philadelphia',
        font : '24px Helvetica',
        fillColor : Cesium.Color.YELLOW,
        //  动态颜色变化
        // fillColor : new Cesium.CallbackProperty(function() {
        //     return Cesium.Color.fromRandom();
        // }, false),
        outlineColor : Cesium.Color.BLACK,
        outlineWidth : 2,
        style : Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin : Cesium.VerticalOrigin.BOTTOM, // 标签底部对齐
        heightReference : Cesium.HeightReference.CLAMP_TO_GROUND, // 贴地
        //设置背景
        backgroundColor : new Cesium.Color(0.165, 0.165, 0.165, 0.8), // 灰色半透明背景
        backgroundPadding : new Cesium.Cartesian2(7, 5), // 背景填充大小
        //缩放
         scale : 1.5, // 放大 1.5 倍

    })
});
```
## 三、几何体
  ### 1. [椭圆 圆](http://www.cesium.xin/cesium/cn/Documentation1.62/EllipseGraphics.html?classFilter=EllipseGraphics) EllipseGraphics
      EllipseGraphics，描述：由中心点，半长轴和半短轴定义的椭圆。椭圆符合地球的曲率，可以放置在表面或可以选择将其挤出成一定体积。中心点由包含的 Entity 确定。
      Cesium.EllipseGeometry 和 Cesium.EllipseGraphics 都用于在 Cesium 中创建椭圆或圆形，但它们的用法和适用场景有所不同。它们的主要区是：
 Cesium.EllipseGeometry： 适合在性能要求高或需要对几何数据进行复杂操作的场景中使用。不直接显示，通常与 Cesium.Primitive 结合使用。
```js
    // 创建椭圆几何体
    let ellipseGeometry = new Cesium.EllipseGeometry({
        center: Cesium.Cartesian3.fromDegrees(-100.0, 40.0),
        semiMajorAxis: 500000.0,
        semiMinorAxis: 300000.0,
    });

    // 创建几何实例
    let geometryInstance = new Cesium.GeometryInstance({
        geometry: ellipseGeometry,
        attributes: {
            color: Cesium.ColorGeometryInstanceAttribute.fromColor(Cesium.Color.BLUE)
        }
    });

    // 使用 Primitive 将几何体添加到场景中
    viewer.scene.primitives.add(new Cesium.Primitive({
        geometryInstances: geometryInstance,
        appearance: new Cesium.PerInstanceColorAppearance()
    }));
```
Cesium.EllipseGraphics： 适合快速创建、显示和管理实体，通常与 Cesium.Entity 结合使用。提供了更易于使用的属性，如颜色、材质、贴地等。
```js
// 创建并显示一个椭圆实体
viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(-100.0, 40.0),
    ellipse: {
        semiMajorAxis: 500000.0,
        semiMinorAxis: 300000.0,
        material: Cesium.Color.BLUE.withAlpha(0.5),
        height: 0,  // 设置高度
        extrudedHeight: 100000.0,  // 拉伸高度
    }
});

// 聚焦到实体
viewer.zoomTo(viewer.entities);

```
Cesium.EllipseGeometry 更适合对几何体数据有高性能需求或需要复杂控制的场景。Cesium.EllipseGraphics 则是更高层的抽象，适合快速开发、易于使用的场景。
  ### 2. [球 ⚽️](http://www.cesium.xin/cesium/cn/Documentation1.62/EllipsoidGraphics.html?classFilter=EllipsoidGraphics) EllipsoidGraphics
是不是定眼一看 球的类和椭圆的类一样， **认真看** 不是一样的 **EllipsoidGraphics**（球） **EllipseGraphics** （圆）, 当然英语好的一眼就看出来了，很显然我不是 四级都考了两遍还没过的我当然看不出来，咱们看中文可以看出来 一个是**球**三维的 一个是**形**二维的 所以：
```text
EllipsoidGraphics 适用于创建三维的椭球形或球体模型，通常用于天体、行星等三维物体的表示。
EllipseGraphics 则适用于创建二维的椭圆或圆形图形，常用于表示地图上的区域、范围等平面图形。
```
```js
// 创建并显示一个球体
viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(-100.0, 40.0),
    ellipsoid: {
        radii: new Cesium.Cartesian3(500000.0, 500000.0, 300000.0),
        material: Cesium.Color.BLUE.withAlpha(0.5),
        outline: true,
        outlineColor: Cesium.Color.BLACK
    }
});
```


  ### 3. [圆柱](http://www.cesium.xin/cesium/cn/Documentation1.62/CylinderGraphics.html?classFilter=CylinderGraphics) CylinderGraphics
      圆柱和椭圆一样也有两个类，和上面👆类似所以后续只介绍常用的， *Cesium.CylinderGraphics* ， Cesium.CylinderGeometry
Cesium.CylinderGraphics 更常用于快速创建和管理场景中的实体对象，通过简单的配置即可在场景中显示圆柱体。
```js
// 创建并显示一个圆柱实体
viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(-100.0, 40.0),
    cylinder: new Cesium.CylinderGraphics({
        length: 400000.0,  // 圆柱长度
        topRadius: 200000.0,  // 顶部半径
        bottomRadius: 200000.0,  // 底部半径
        material: Cesium.Color.BLUE.withAlpha(0.5),  // 材质和颜色
        //  heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND // 相对地形
        // heightReference: Cesium.HeightReference.CLAMP_TO_GROUND // 贴地
    })
});
```
  ### 4. [矩形](http://www.cesium.xin/cesium/cn/Documentation1.62/RectangleGraphics.html?classFilter=RectangleGraphics) RectangleGraphics
      常用类 Cesium.RectangleGraphics， 用于在场景中直接创建和显示矩形实体。它是 Cesium.Entity 的一部分，提供了更高级的 API 用于简单的矩形绘制和管理。   高级类 Cesium.RectangleGeometry 
```js
viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(-100.0, 40.0),
    rectangle: {
        coordinates: Cesium.Rectangle.fromDegrees(-80.0, 30.0, -70.0, 40.0),
        material: Cesium.Color.GREEN.withAlpha(0.6),
        outline: true,
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 2,
        heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND // 相对地形
    }
});
```
  ### 5. [墙壁](http://www.cesium.xin/cesium/cn/Documentation1.62/WallGraphics.html?classFilter=WallGraphics) WallGraphics
        Cesium.WallGraphics 描述定义为线带和可选的最大和最小高度的二维墙。墙符合地球仪的曲率，可以沿着地面或在高处放置。 高级类 WallGeometry
```js
viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(-100.0, 40.0),
    wall: {
        positions: Cesium.Cartesian3.fromDegreesArrayHeights([
            -115.0, 50.0, 100000.0,
            -110.0, 50.0, 200000.0
        ]),
        material: Cesium.Color.YELLOW.withAlpha(0.5)
    }
});
```
  ### 6. [管道](https://cesium.xin/cesium/cn/Documentation1.62/CorridorGraphics.html?classFilter=CorridorGraphics) CorridorGraphics
        Cesium.CorridorGraphics 描述走廊，走廊是由中心线和宽度定义的形状符合地球的曲率。它可以放置在地面上或高空并可以选择挤出成一个体积。 高级类CorridorGeometry
```js
viewer.entities.add({
    corridor: {
        positions: Cesium.Cartesian3.fromDegreesArray([
            -100.0, 40.0,
            -110.0, 45.0
        ]),
        width: 200000.0,
        material: Cesium.Color.ORANGE.withAlpha(0.5)
    }
});
```

  ### 7. [模型](http://www.cesium.xin/cesium/cn/Documentation1.62/ModelGraphics.html?classFilter=ModelGraphics) ModelGraphics
      ModelGraphics 是 Cesium 中用于加载和管理 3D 模型的高级接口，支持广泛的模型格式和渲染选项。属性可以控制模型的缩放、颜色、动画、阴影、轮廓、混合模式等，提供了丰富的自定义选项。
```js
let viewer = new Cesium.Viewer('cesiumContainer');

// 加载并显示一个 3D 模型
let modelEntity = viewer.entities.add({
    name: "3D Model",
    position: Cesium.Cartesian3.fromDegrees(-123.0744619, 44.0503706, 0),
    model: {
        uri: "path/to/model.glb", // 通常是 glTF 或 GLB 文件的路径。这个路径可以是本地路径或者远程服务器上的 URL。
        scale: 2.0, // 模型的缩放比例。默认为 1.0，表示模型以其原始大小显示。
        minimumPixelSize: 128, // 模型在视口中的最小像素大小。即使模型距离视角很远，也会保持该最小大小。
        maximumScale: 200.0, // 限制模型的最大缩放比例，用于防止模型放大过多。
        silhouetteColor: Cesium.Color.RED, // 模型轮廓的颜色。用于在模型的边缘绘制轮廓线。
        silhouetteSize: 3.0, // 模型轮廓的大小（单位为像素）。
        color: Cesium.Color.BLUE.withAlpha(0.5), // 应用于模型的颜色。通常用于着色模型或覆盖其原始纹理。
        colorBlendMode: Cesium.ColorBlendMode.MIX, // 指定颜色如何与模型的原始纹理混合。可以是 Cesium.ColorBlendMode.HIGHLIGHT、Cesium.ColorBlendMode.REPLACE 或 Cesium.ColorBlendMode.MIX。
        colorBlendAmount: 0.5, // 控制颜色混合的比例，用于 MIX 模式下。范围从 0.0 到 1.0。
        shadows: Cesium.ShadowMode.ENABLED, // 控制模型是否投射或接收阴影。可以是 Cesium.ShadowMode.ENABLED、Cesium.ShadowMode.DISABLED 或 Cesium.ShadowMode.CAST_ONLY。
        distanceDisplayCondition: new Cesium.DistanceDisplayCondition(10.0, 10000.0), // 控制模型在一定距离范围内可见。指定近距离和远距离的阈值。（模型在 10 到 1000 米范围内可见）
        heightReference: Cesium.HeightReference.CLAMP_TO_GROUND, // 控制模型的高度基准。可以是 Cesium.HeightReference.NONE、Cesium.HeightReference.CLAMP_TO_GROUND 或 Cesium.HeightReference.RELATIVE_TO_GROUND。
        runAnimations: true // 布尔值，控制模型中的动画是否自动运行。默认为 true。
    }
});

// 聚焦到模型位置
viewer.zoomTo(modelEntity);
```
  ### 8. [多边形](http://www.cesium.xin/cesium/cn/Documentation1.62/PolygonGraphics.html?classFilter=PolygonGraphics) PolygonGraphics
        PolygonGraphics是 Cesium 中用于创建和管理多边形实体的类。它允许你在 3D 地图或场景中绘制具有不同属性和样式的多边形。
```js
var viewer = new Cesium.Viewer('cesiumContainer');

// 定义多边形的顶点
var positions = Cesium.Cartesian3.fromDegreesArray([
    -115.0, 37.0,
    -115.0, 32.0,
    -107.0, 33.0,
    -102.0, 31.0,
    -102.0, 35.0
]);

// 添加一个多边形实体
var polygonEntity = viewer.entities.add({
    name: "Red polygon",
    polygon: {
        hierarchy: new Cesium.PolygonHierarchy(positions), // 多边形的边界定义，通常是一个或多个 Cesium.Cartesian3 数组，用于指定多边形的顶点。
        height: 1000.0, //  多边形的高度（相对于地球表面），用于使多边形悬浮在某一高度。
        extrudedHeight: 2000.0, // 多边形的拉伸高度（也称为挤出高度），使其成为一个 3D 的体积。
        material: Cesium.Color.RED.withAlpha(0.5), // 半透明红色 --多边形的填充材质，支持颜色、图像、视频、网格等多种材质类型。
        outline: true, // 显示轮廓
        outlineColor: Cesium.Color.BLACK, // 黑色轮廓线
        stRotation: Cesium.Math.toRadians(45.0), // 多边形的纹理旋转角度（弧度），用于控制纹理的旋转。
        perPositionHeight: true, // 布尔值，是否使用每个顶点的高度，允许多边形的每个顶点具有不同的高度。
        closeTop: false, // 布尔值，分别用于指定是否关闭拉伸多边形的顶部和底部。
        closeBottom: true, // 布尔值，分别用于指定是否关闭拉伸多边形的顶部和底部。
        distanceDisplayCondition: new Cesium.DistanceDisplayCondition(10.0, 100000.0), // 控制模型在一定距离范围内可见。指定近距离和远距离的阈值。（模型在 10 到 10000 米范围内可见）
        classificationType: Cesium.ClassificationType.BOTH  // 指定多边形用于地形、3D 瓦片或地形和 3D 瓦片的分类。可以是 Cesium.ClassificationType.TERRAIN 或 Cesium.ClassificationType.CESIUM_3D_TILE。
    }
});
// 聚焦到多边形位置
viewer.zoomTo(polygonEntity);
```
其中 **classificationType** 这个参数有点难以理解，但是很多场景下会使用到 所以多解释一下
```text
classificationType 主要有以下几个值：

Cesium.ClassificationType.TERRAIN:
  多边形将对地形进行分类。多边形将影响地形的外观，例如改变地形颜色或显示遮挡区域。
  使用场景: 当你想要在地形上显示区域的覆盖或遮挡，例如显示建筑区域、进行土地利用分类、绘制禁飞区等。

Cesium.ClassificationType.CESIUM_3D_TILE:
  多边形将对 3D 瓦片进行分类。这意味着多边形会影响 3D 瓦片（如建筑物、3D 模型等）的外观。
  使用场景: 当你需要在 3D 城市模型中标识特定建筑物、区域或执行分析，如热图覆盖、选定建筑高亮等。

Cesium.ClassificationType.BOTH:
  多边形将同时对地形和 3D 瓦片进行分类。它将同时影响地形和 3D 瓦片的外观。
  使用场景: 在需要同时影响地形和 3D 瓦片的情况下使用，如在一个复杂的三维场景中同时显示区域分析和建筑物高亮。

```

### 标签
可以通过结合 Entity 类的 Billboard、Label 和 Polyline 等实体来实现。
```js
var viewer = new Cesium.Viewer('cesiumContainer');

// 定义坐标
var longitude = 120.0;
var latitude = 20.0;
var height = 100.0; // 对话框的高度（相对于地面）
var groundHeight = 0.0; // 地面高度

// 创建对话框和箭头
var entities = viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(longitude, latitude, height),
    billboard: {
        image: 'path/to/dialogue-box.png',
        width: 100,
        height: 60,
    },
    polyline: {
        positions: Cesium.Cartesian3.fromDegreesArrayHeights([
            longitude, latitude, height - 30, // 对话框的底部中心
            longitude, latitude, groundHeight // 地面上的位置
        ]),
        width: 2,
        material: Cesium.Color.WHITE
    },
    label: {
        text: 'Your text here',
        font: '14pt sans-serif',
        fillColor: Cesium.Color.BLACK,
        showBackground: false,
        verticalOrigin: Cesium.VerticalOrigin.TOP,
        pixelOffset: new Cesium.Cartesian2(0, -30) // 偏移量以确保文本在对话框内
    }
});

// 聚焦到该位置
viewer.zoomTo(entities);

```

## 四、特殊标会
### 1.扫描波
假设场景：在3D地球上有个卫星，从卫星上发射一个圆柱形波扫描地球这个卫星按照规定轨迹在航行
* 我们用到上述所了解到的 圆柱形（CylinderGraphics）加上添加卫星模型（ModelGraphics）这两个类实现
* **注意** model的路径key值是 **uri**  并不是传统的 url 所以要细，因为一个字母死活找不到bug的情况常常发生！！
```js
const viewer = new Cesium.Viewer("cesiumContainer");
const entities = viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(-100.0, 40.0, 400000),
    cylinder: new Cesium.CylinderGraphics({
        length: 400000.0,  // 圆柱长度
        topRadius: 2000.0,  // 顶部半径
        bottomRadius: 200000.0,  // 底部半径
        material: Cesium.Color.BLUE.withAlpha(0.5),  // 材质和颜色
        heightReference: Cesium.HeightReference.CLAMP_TO_GROUND // 贴地
    }),
    model: {
      uri: '../SampleData/models/CesiumAir/Cesium_Air.glb',
      scale: 2.0, // 模型的缩放比例。默认为 1.0，表示模型以其原始大小显示。
      minimumPixelSize: 128, // 模型在视口中的最小像素大小。即使模型距离视角很远，也会保持该最小大小。
      maximumScale: 200.0, // 限制模型的最大缩放比例，用于防止模型放大过多。
    }
});
viewer.zoomTo(entities);

// 可以个给个定时器让飞机动一动，速度根据 +0.02经纬度 和 定时器时间控制 
let currentIndex = 0;
let flag = true;
// 每隔一段时间更新实体位置
setInterval(function() {
    flag ? currentIndex += 0.02 : currentIndex -= 0.02
    if(currentIndex > 50) flag = false
    if(currentIndex < 0) flag = true
    let position = Cesium.Cartesian3.fromDegrees(120.1884654 + currentIndex, 14.152481, 4020);
    entities.position = position;
}, 1000); // 每隔100毫秒（10/1秒）更新位置

viewer.zoomTo(entities);

```

其中如果想修改 圆柱/圆锥体的表面颜色或者材质可以修改 **material** 参数
 - 颜色材质 (ColorMaterialProperty)
```js
material: Cesium.Color.RED.withAlpha(0.5) // 半透明红色
```
 - 图像材质 (ImageMaterialProperty)
```js
material: new Cesium.ImageMaterialProperty({
    image: 'path/to/image.png',
    repeat: new Cesium.Cartesian2(2.0, 2.0) // 控制图像重复
})
``` 
 - 网格材质 (GridMaterialProperty)
```js
material: new Cesium.GridMaterialProperty({
    color: Cesium.Color.YELLOW,
    cellAlpha: 0.2, // 网格线透明度
    lineCount: new Cesium.Cartesian2(8, 8), // 网格线数量
    lineThickness: new Cesium.Cartesian2(2.0, 2.0) // 网格线粗细
})
```
 - 条纹材质 (StripeMaterialProperty)
```js
material: new Cesium.StripeMaterialProperty({
    evenColor: Cesium.Color.WHITE,
    oddColor: Cesium.Color.BLUE,
    repeat: 5, // 条纹重复次数
    offset: 0.5, // 条纹偏移量
    orientation: Cesium.StripeOrientation.HORIZONTAL // 条纹方向
})
```
 - 波浪材质 (CheckerboardMaterialProperty)
```js
material: new Cesium.CheckerboardMaterialProperty({
    evenColor: Cesium.Color.WHITE,
    oddColor: Cesium.Color.BLACK,
    repeat: new Cesium.Cartesian2(4, 4) // 棋盘格数量
})
```
 - 发光材质 (PolylineGlowMaterialProperty)
```js
material: new Cesium.PolylineGlowMaterialProperty({
    glowPower: 0.2, // 发光强度
    color: Cesium.Color.BLUE
})
```
 - 水波材质 (WaterMaterialProperty)
```js
material: new Cesium.WaterMaterialProperty({
    baseWaterColor: Cesium.Color.BLUE,
    specularIntensity: 0.5,
    normalMap: 'path/to/waterNormal.png'
})
```

上会说到咱们简单的说了一些基础知识，这些基本上后面常常用到。接下来上一点难度的，我们只是在地图上添加实体，增删改查只做了一个还有三没写呢

### 2.标会编辑
在修改之前呢还有几个问题如何让鼠标点击生效，怎么给实体添加事件监听？地图上开始修改对应的参数？












空间网

雷达

动态线

圆锥沿轴旋转

雷达扫描波

标牌

图谱

场景标会保存导出导入

画圈范围 监听目标是否进入

放大镜

固定视角动画

测量距离

测量面积

扇形

扩散波

扫描波

仪表盘

标会编辑

贴地扩散圆

点标记

扩散圆

扩散圆多个





# Vue 3 + Vite

├── node_modules/
├── public/
│   └── favicon.ico
├── src/
│   ├── assets/
│   │   └── logo.png
│   ├── components/
│   │   └── CesiumMap.vue
│   ├── stores/
│   │   └── counter.js
│   ├── App.vue
│   ├── main.js
│   ├── router/
│   │   └── index.js
│   └── style.css
├── .gitignore
├── index.html
├── package.json
├── vite.config.js
└── README.md

