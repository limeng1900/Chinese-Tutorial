---
title: Analyze data with Turf.js and Mapbox.js
description: Using Turf.js, you can add some spatial analysis to your map to solve problems. This guide walks through an example of Turf.js and Mapbox.js in a real-world context.
thumbnail: analysisWithTurfMapboxJs
level: 2
topics:
- web apps
- analysis
language:
- JavaScript
prereq: Familiarity with front-end development concepts.
legacy: true
prependJs:
  - "import * as constants from '../../constants';"
  - "import Note from '@mapbox/dr-ui/note';"
  - "import BookImage from '@mapbox/dr-ui/book-image';"
  - "import { LegacyNote } from '../../components/legacy-note';"
  - "import { hospitalsLex } from '../../snippets/hospitals-lex';"
  - "import { librariesLex } from '../../snippets/libraries-lex';"
  - "import UserAccessToken from '../../components/user-access-token';"
  - "import DemoIframe from '@mapbox/dr-ui/demo-iframe';"
contentType: tutorial
---

{{
  <LegacyNote
    legacyProduct='Mapbox.js'
    alternativeResource={{
      title: 'Analyze data with Turf.js and Mapbox GL JS',
      link: '/help/tutorials/analysis-with-turf/'
    }}
  />
}}

本教程将带你了解如何使用 [Turf.js](http://turfjs.org)，它是一个用来空间分析和统计的 JavaScript 库。

假设你是肯塔基州列克星敦市图书馆健康与安全管理团队的一员，为了应对万一的紧急情况，你工作任务中的一个重要部分是了解 __对于每个图书馆来说哪个医院最近__ 。下面这个示例将向你展示图书馆和医院的地图，当点击其中一个图书馆时，地图会展示出离它最近的医院。

{{
  <DemoIframe gl={false} src="/help/demos/turfjs-intro-js/demo-five.html" />
}}

## 起步

开始之前，这里有一些你需要的资源：

- [__一个 tileset ID__](/help/glossary/tileset-id) 一个指向你在 Mapbox 上创建的唯一地图的 ID。
- [__一个 access token__](/help/glossary/access-token/) 用来将地图关联到你的账户。

```js
L.mapbox.accessToken = '{{ <UserAccessToken /> }}';
```

- [__Mapbox.js__](https://www.mapbox.com/mapbox.js/) 它是用于创建地图的 Mapbox JavaScript API。
- [__Turf.js__](http://turfjs.org/) 它是你今天用来给地图添加分析功能的 JavaScript 库。
- __数据__  这个示例用到了两个数据文件：列克星敦的医院数据（hospitals in Lexington）和 列克星敦的图书馆数据（KY and libraries in Lexington, KY）。
- __一个文本编辑器__  你需要用它来编写HTML、CSS 和 JavaScript。

## 添加主体框架

本教程中要引入最新版本的 Mapbox.js 和 Turf.js。在文本编辑器中创建一个新的 HTML 文件，复制以下代码，将这些库添加到你的 head 中：

```html
<link href='https://api.mapbox.com/mapbox.js/{{constants.VERSION_MAPBOXJS}}/mapbox.css' rel='stylesheet' />
<script src='https://api.mapbox.com/mapbox.js/{{constants.VERSION_MAPBOXJS}}/mapbox.js'></script>
<script src='https://api.mapbox.com/mapbox.js/plugins/turf/{{constants.VERSION_TURF}}/turf.min.js'></script>
```

现在，添加你的地图元素，首先在 `body` 中为你的地图创建一个空 `div`：

```html
<div id='map'></div>
```

接下来，向 `head` 中的 `style` 元素添加一些 CSS，让地图的宽度充满页面。

```css
body {
  margin: 0;
  padding: 0;
}

#map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}
```

## 地图初始化

现在你的页面已经有了合适的框架，下一步使用 Mapbox.js 来加载一副地图。

你将创建一个名称为 `map` 的 `L.mapbox.map` 对象，并使用 `setView` 将地图中心定位于列克星敦市。这里就是你要用到access token 和 tileset ID 的地方。请在 `<body>` 中上述 HTML 的后面添加以下代码：

```js
L.mapbox.accessToken = '{{ <UserAccessToken /> }}';
var map = L.mapbox.map('map')
  .setView([38.05, -84.5], 12)
  .addLayer(L.mapbox.styleLayer('mapbox://styles/mapbox/light-v10'));
map.scrollWheelZoom.disable();
```

{{
  <DemoIframe gl={false} src="/help/demos/turfjs-intro-js/demo-one.html" />
}}

很好，现在你的页面有了一个以列克星敦为中心的地图。

{{
  <Note imageComponent={<BookImage />}>
    <p>上面代码中额外有一行代码关闭了地图滚轮缩放功能，这个是可选的。</p>
  </Note>
}}


## 加载数据

之前提到，这个示例使用了两个数据文件：列克星敦的医院数据（hospitals in Lexington）和 列克星敦的图书馆数据（libraries in Lexington, KY），它们的形式都是 GeoJSON FeatureCollection。下一步，将它们作为 `L.mapbox.featureLayer` 对象加入地图，并且添加一些代码来让它们拥有不同的样式。另外，还要通过地图边界对要素的适应调整，从而确保所有的点都包含在地图的视图范围内。

```js
L.mapbox.accessToken = '{{ <UserAccessToken /> }}';

// 将GeoJSON对象存入变量
var hospitals = {{ hospitalsLex }};
var libraries = {{ librariesLex }};

// 给医院 GeoJSON 添加标记的颜色、符号和尺寸设置
for (var i = 0; i < hospitals.features.length; i++) {
  hospitals.features[i].properties['marker-color'] = '#DC143C';
  hospitals.features[i].properties['marker-symbol'] = 'hospital';
  hospitals.features[i].properties['marker-size'] = 'small';
}

// 给图书馆 GeoJSON 添加标记的颜色、符号和尺寸设置
for (var j = 0; j < libraries.features.length; j++) {
  libraries.features[j].properties['marker-color'] = '#4169E1';
  libraries.features[j].properties['marker-symbol'] = 'library';
  libraries.features[j].properties['marker-size'] = 'small';
}

var map = L.mapbox.map('map')
  .setView([38.05, -84.5], 12)
  .addLayer(L.mapbox.styleLayer('mapbox://styles/mapbox/light-v10'));
map.scrollWheelZoom.disable();

var hospitalLayer = L.mapbox.featureLayer(hospitals)
  .addTo(map);
var libraryLayer = L.mapbox.featureLayer(libraries)
  .addTo(map);

// 在地图加载时将地图缩放到 libraryLayer 要素
map.fitBounds(libraryLayer.getBounds());
```

请注意在你创建 `map` 对象 _之后_ 才能定义`hospitalLayer` 和 `libraryLayer`。你必须按照这个顺序才能确保它们可以加载到地图上。

{{
  <DemoIframe gl={false} src="/help/demos/turfjs-intro-js/demo-two.html" />
}}

另外，可以将 GeoJSON 保存为 `.geojson` 文件并且[将这些文件加载到地图](https://www.mapbox.com/mapbox.js/example/v1.0.0/geojson-marker-from-url/)。如果要这么做，就需要通过本地 web 服务来运行这个应用，否则会出现[跨域资源共享](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) 错误。

## 添加交互

你的用户可能会想要知道地图上显示的图书馆和医院的名称，所以下面我们来添加弹窗。当用户悬停在标记上时，给要素添加弹窗。在你创建的 `hospitalLayer` 和 `libraryLayer` 后面插入以下内容：

```js
//  给 hospitalLayer 和 libraryLayer 的每个要素绑定一个弹窗
hospitalLayer.eachLayer(function(layer) {
  layer.bindPopup('<strong>' + layer.feature.properties.Name + '</strong>', { closeButton: false });
}).addTo(map);
libraryLayer.eachLayer(function(layer) {
  layer.bindPopup(layer.feature.properties.Name, { closeButton: false });
}).addTo(map);

// 悬停时打开弹窗
libraryLayer.on('mouseover', function(e) {
  e.layer.openPopup();
});
hospitalLayer.on('mouseover', function(e) {
  e.layer.openPopup();
});
```

{{
  <DemoIframe gl={false} src="/help/demos/turfjs-intro-js/demo-three.html" />
}}

接下来，你将通过添加一些分析功能，来让列克星敦的图书馆和医院地图更加实用。

## 使用 Turf

[Turf](http://turfjs.org) 是一个给 web 地图增加空间和统计分析功能的 JavaScript 库。它除了包含了许多常用的 GIS 工具（如缓冲区、叠加和合并），还拥有许多统计分析函数（如加和、求中位数和均值）。

幸运的是，Turf 有一些函数可以在这里帮到你！你可以升级你的地图，从而在用户点击图书馆时向他展示哪个医院离得最近。

作为第一步，在有人点击图书馆标记时创建一个“事件处理程序”。当事件发生时，例如点击了一个标记，事件处理程序将告诉地图如何响应。之前，我们已经创建了一个医院和图书馆标记悬停的事件处理程序，现在需要继续为点击添加一个事件处理程序。

```js
libraryLayer.on('click', function(e) {});
```

这是一个事件处理程序的构造，你要将任何想要在点击后发生的内容放进大括号 `{}` 中。在这个例子中，你需要使用 Turf 来识别出距离被点击图书馆最近的医院，并且将它的标记放大。

```js
libraryLayer.on('click', function(e) {
  // 从 libraryFeatures 和 hospitalFeatures 获得 GeoJSON
  var libraryFeatures = libraryLayer.getGeoJSON();
  var hospitalFeatures = hospitalLayer.getGeoJSON();

  // 使用 Turf 找到离被点击图书馆最近的医院
  var nearestHospital = turf.nearest(e.layer.feature, hospitalFeatures);

  // 将这个最近医院的标记尺寸修改为大号
  nearestHospital.properties['marker-size'] = 'large';

  // 将新的 GeoJSON 添加到 hospitalLayer
  hospitalLayer.setGeoJSON(hospitalFeatures);
});
```

{{
  <DemoIframe gl={false} src="/help/demos/turfjs-intro-js/demo-four.html" />
}}

很棒！ 差不多快要完成了。

## 收尾工作

<!--copyeditor ignore previously-->

当用户点击图书馆时，最近的医院就会变大。但是当用户点击另一个图书馆或地图时，之前这个最近的医院不会恢复成小标记。解决这个问题的同时，我们再加一些代码让最近医院在标记变大时打开弹窗。

在点击事件处理程序之前加入下面的函数。

```js
// 将标记尺寸重置为小号
function reset() {
  var hospitalFeatures = hospitalLayer.getGeoJSON();
  for (var i = 0; i < hospitalFeatures.features.length; i++) {
    hospitalFeatures.features[i].properties['marker-size'] = 'small';
  }
  hospitalLayer.setGeoJSON(hospitalFeatures);
}
```

然后在点击处理中添加一个函数调用 `reset()`，再加一些代码来让最近医院的标记在变大时打开弹窗。

```js
libraryLayer.on('click', function(e) {
  // 将所有标记尺寸重置为小号
  reset();

  // 从 libraryFeatures 和 hospitalFeatures 获得 GeoJSON
  var libraryFeatures = libraryLayer.getGeoJSON();
  var hospitalFeatures = hospitalLayer.getGeoJSON();

  // 使用 Turf 找到离被点击图书馆最近的医院
  var nearestHospital = turf.nearest(e.layer.feature, hospitalFeatures);

  // 将这个最近医院的标记尺寸修改为大号
  nearestHospital.properties['marker-size'] = 'large';

  // 将新的 GeoJSON 添加到 hospitalLayer
  hospitalLayer.setGeoJSON(hospitalFeatures);

  // 给新的 hospitalLayer 绑定弹窗，并将最近医院的弹窗打开
  hospitalLayer.eachLayer(function(layer) {
    layer.bindPopup('<strong>' + layer.feature.properties.Name + '</strong>', { closeButton: false });
    if (layer.feature.properties['marker-size'] === 'large') {
      layer.openPopup();
    }
  });
});
```

最后，在末尾添加一点代码，使得在点击地图上任意位置（除了图书馆）时，将所有的标记尺寸重置为小号。

```js
map.on('click', function(e) {
  reset();
});
```

{{
  <DemoIframe gl={false} src="/help/demos/turfjs-intro-js/demo-five.html" />
}}

## 完成

干得漂亮！你已经成功创建了一个地图，它可以动态地计算对于每个图书馆哪个医院离得最近。你完成的 HTML 文件应该是这样的：

```html
<html>
<head>
<meta charset=utf-8 />
<title>Turf.js Map</title>
<meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
<script src='https://api.mapbox.com/mapbox.js/{{constants.VERSION_MAPBOXJS}}/mapbox.js'></script>
<link href='https://api.mapbox.com/mapbox.js/{{constants.VERSION_MAPBOXJS}}/mapbox.css' rel='stylesheet' />
<script src='https://api.mapbox.com/mapbox.js/plugins/turf/{{constants.VERSION_TURF}}/turf.min.js'></script>
<style>
  body {
    margin: 0;
    padding: 0;
  }

  #map {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
  }
</style>
</head>
<body>
  <div id='map'></div>
  <script>
    L.mapbox.accessToken = '{{ <UserAccessToken /> }}';

    var hospitals = {{ hospitalsLex }};
    var libraries = {{ librariesLex }};

    // 给医院 GeoJSON 添加标记的颜色、符号和尺寸设置
    for (var i = 0; i < hospitals.features.length; i++) {
      hospitals.features[i].properties['marker-color'] = '#DC143C';
      hospitals.features[i].properties['marker-symbol'] = 'hospital';
      hospitals.features[i].properties['marker-size'] = 'small';
    }

    // 给图书馆 GeoJSON 添加标记的颜色、符号和尺寸设置
    for (var j = 0; j < libraries.features.length; j++) {
      libraries.features[j].properties['marker-color'] = '#4169E1';
      libraries.features[j].properties['marker-symbol'] = 'library';
      libraries.features[j].properties['marker-size'] = 'small';
    }

    var map = L.mapbox.map('map')
      .setView([38.05, -84.5], 12)
      .addLayer(L.mapbox.styleLayer('mapbox://styles/mapbox/light-v10'));
    map.scrollWheelZoom.disable();

    var hospitalLayer = L.mapbox.featureLayer(hospitals)
      .addTo(map);
    var libraryLayer = L.mapbox.featureLayer(libraries)
      .addTo(map);

    map.fitBounds(libraryLayer.getBounds());

    // 给 hospitalLayer 和 libraryLayer 的每个要素绑定一个弹窗
    hospitalLayer.eachLayer(function(layer) {
      layer.bindPopup('<strong>' + layer.feature.properties.Name + '</strong>', { closeButton: false });
    }).addTo(map);
    libraryLayer.eachLayer(function(layer) {
      layer.bindPopup(layer.feature.properties.Name, { closeButton: false });
    }).addTo(map);

    // 悬停时打开弹窗
    libraryLayer.on('mouseover', function(e) {
      e.layer.openPopup();
    });
    hospitalLayer.on('mouseover', function(e) {
      e.layer.openPopup();
    });

    // 将标记尺寸重置为小号
    function reset() {
      var hospitalFeatures = hospitalLayer.getGeoJSON();
      for (var k = 0; k < hospitalFeatures.features.length; k++) {
        hospitalFeatures.features[k].properties['marker-size'] = 'small';
      }
      hospitalLayer.setGeoJSON(hospitalFeatures);
    }

    // 当图书馆被点击时，操作如下
    libraryLayer.on('click', function(e) {
      // 将所有标记尺寸重置为小号
      reset();
      // 从 libraryFeatures 和 hospitalFeatures 获得 GeoJSON
      var libraryFeatures = libraryLayer.getGeoJSON();
      var hospitalFeatures = hospitalLayer.getGeoJSON();
      // 使用 Turf 找到离被点击图书馆最近的医院
      var nearestHospital = turf.nearest(e.layer.feature, hospitalFeatures);
      // 将这个最近医院的标记尺寸修改为大号
      nearestHospital.properties['marker-size'] = 'large';
      // 将新的 GeoJSON 添加到 hospitalLayer
      hospitalLayer.setGeoJSON(hospitalFeatures);
      // 给新的 hospitalLayer 绑定弹窗，并将最近医院的弹窗打开
      hospitalLayer.eachLayer(function(layer) {
        layer.bindPopup('<strong>' + layer.feature.properties.Name + '</strong>', { closeButton: false });
        if (layer.feature.properties['marker-size'] === 'large') {
          layer.openPopup();
        }
      });
    });

    // 点击地图上任意位置时，将所有的标记尺寸重置为小号
    map.on('click', function(e) {
      reset();
    });

  </script>
</body>
</html>
```

## 下一步

[Turf](http://turfjs.org) 拥有几十种工具可以帮你进一步扩展这个地图。比如，你可以使用[turf.distance](http://turfjs.org/docs/#distance) ，不只判断哪个医院最近，还能明确它到底有多远。Turf.js 可以为你带来无限可能！
