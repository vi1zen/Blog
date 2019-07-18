---
title: 百度地图覆盖物Marker的一些坑
date: 2018.03.14 10:16:23
categories:
 - Android
tags: 
 - 百度地图
 
---
### 引言

最近项目在用百度地图SDK进行开发，在添加覆盖物的时候遇到了一些坑，在此记录下。

### 添加Marker

添加Marker有两种方式，一种是通过BaiduMap的addOverlays()方法添加，另一种是通过OverlayManager的addToMap()添加，实际上内部实现都是一样的，只是API不同

- 通过BaiduMap.addOverlays()方法添加覆盖物
```java
OverlayOptions option = new MarkerOptions()；
option.position(latLng).icon(unselectedBitmap).title(cardStatistics.getCusName()).extraInfo(bundle);
baiduMap.addOverlays(overlayOptions);
```
- 通过OverlayManager的addToMap()方法添加覆盖物
```java
OverlayManager overlayManager = new OverlayManager(mapView.getMap()) {
    @Override
    public List<OverlayOptions> getOverlayOptions() {
        return overlayOptions;
    }

    @Override
    public boolean onMarkerClick(Marker marker) {
        onMarkerClick(marker);
        return false;
    }

    @Override
    public boolean onPolylineClick(Polyline polyline) {
        return false;
    }
};
overlayManager.addToMap();
```

<!-- more -->


### 获取全部Marker

- 通过addOverlays()返回值获取

```
List<Overlay> overlays = baiduMap.addOverlays(overlayOptions);
```

- 通过反射获取OverlayManager的mOverlayList字段获取

```java
try {
    Field field = OverlayManager.class.getDeclaredField("mOverlayList");
    field.setAccessible(true);
    List<Overlay> overlays = (List<Overlay>) field.get(overlayManager);
    ...
} catch (NoSuchFieldException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
}
```

### 使所有覆盖物Marker合适的显示在同一屏中

- 使用OverlayManager的zoomToSpan()方法

```java
overlayManager.zoomToSpan();
```

- 自己添加zoomToSpan()方法

```java
/**
 * 缩放地图，使所有Overlay都在合适的视野内
 * 注： 该方法只对Marker类型的overlay有效
 */
public void zoomToSpan() {
    if (overlays != null && overlays.size() > 0) {
        LatLngBounds.Builder builder = new LatLngBounds.Builder();
        for (Overlay overlay : overlays) {
            // polyline 中的点可能太多，只按marker 缩放
            if (overlay instanceof Marker) {
                builder.include(((Marker) overlay).getPosition());
            }
        }
        mapView.getMap().setMapStatus(MapStatusUpdateFactory
                .newLatLngBounds(builder.build()));
    }
}
```





