---
layout: post
title:  HTML5 使用地理位置定位
date:   2019-04-12 19:00:00 +0800
categories: 开发者
tag: HTML
---

* content
{:toc}

根据用户的地理位置提供相关地区服务，已经是非常普遍的一项功能，例如本地生活类网站、外卖网站等。HTML5 中提供了获取用户位置的能力，使用 HTML5 Geolocation API 来构建基于地理位置的应用。

### geolocation 对象

地理位置 API 通过 navigator.geolocation 提供。如果该对象存在，那么地理位置服务可用。

```js
if ("geolocation" in navigator) {
  /* 地理位置服务可用 */
} else {
  /* 地理位置服务不可用 */
}
```

### getCurrentPosition() 方法

可以调用 `getCurrentPosition()` 函数获取用户当前定位位置。这会异步地请求获取用户位置，并查询定位硬件来获取最新信息。

```js
navigator.geolocation.getCurrentPosition(success, error, options)
```

- `success`: 成功得到位置信息时的回调函数，使用 `Position` 对象作为唯一的参数。 
- `error` 可选: 获取位置信息失败时的回调函数，使用 `PositionError` 对象作为唯一的参数，这是一个可选项。 
- `options` 可选: 一个可选的 `PositionOptions` 对象。

#### Position 对象

`Position` 参数表示在给定的时间的相关设备的位置，其有两个属性：

1. Position.coords 只读: 返回一个定义了当前位置的 Coordinates 对象;
2. Position.timestamp 只读: 返回一个时间戳 DOMTimeStamp， 这个时间戳表示获取到的位置的时间。

[Coordinates](https://developer.mozilla.org/zh-CN/docs/Web/API/Coordinates) 对象包含了很多有用的位置数据信息，主要是以下三个:

- Coordinates.latitude: 坐标纬度;
- Coordinates.longitude: 坐标经度;
- Coordinates.accuracy: 坐标精度，单位为米;
- Coordinates.altitude: 海拔高度(米);
- ...

#### PositionError 对象

`PositionError` 接口包含当定位设备位置时发生错误的信息。

- `PositionError.code` 只读: 返回无符号的、简短的错误码;
1. UNKNOW_ERROR(0)：其他错误;
2. PERMISSION_DENIED(1)：用户拒绝分享位置信息;
3. POSITION_UNAVALABLE(2)：获取用户位置信息失败;
4. TIMEOUT(3)：获取用户位置信息超时;
- `PositionError.message` 只读: 返回一个开发者可以理解的 DOMString 来描述错误的详细信息。

#### PositionOptions

`PositionOptions` 是一个作为 `Geolocation.getCurrentPosition()` 方法以及 `Geolocation.watchPosition()` 方法参数的选项，此选项含有3种可以设置的属性。

- `enableHighAccuracy`: 布尔值，是否获取高精度的位置信息，如果开启可能会增加响应时间，默认值为false。
- `timeout`: 定位超时时间，单位为毫秒，如果到达时间没有取得用户位置信息，则触发失败回调函数，默认值为Infinity，表示无限大。
- `maxinumAge`: 它表明可以返回多长时间（毫秒）内的可获取的缓存位置。，默认值为0，表示设备不能使用一个缓存位置。

### watchPosition()

当用户位置变化时，还可以通过 `watchPosition` 方法监听用户的位置信息，`watchPosition` 的参数和 `getCurrentPosition` 相同。函数执行后返回一个唯一标识，可以通过 `clearWatch` 方法来取消监听。

```js
let watchID = navigator.geolocation.watchPosition(function(position) {
	// ...
});

// 取消监听
navigator.geolocation.clearWatch(watchID);
```

注意: 可以直接调用 `watchPosition()` 函数，不需要先调用 `getCurrentPosition()` 函数。

文档:

- [MDN 使用地理位置定位
](https://developer.mozilla.org/zh-CN/docs/Web/API/Geolocation/Using_geolocation)