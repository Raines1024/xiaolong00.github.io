---
title: 求小块地块中心点的解决思路
date: 2021-03-15 20:01:39
description: 以HikariCP为例
tags: 无人机巡田地块采样点算法开发解决思路
category:
    - 100 学习类
    - 110 编程
    - 111 Java

---

## 背景

单独写一下这个业务的解决思路并不是因为技术上的难以实现，而是做了这个较有感触。

我们经常被外界的各种思路打偏方向，自己的思路跟着别人的思想走下去，没有独立的思考问题而没有自知，这是非常危险的，这件事引起了我的警觉。

先说一下业务上的需求：

在地图上划取一块区域，是为圈地。获取该地的最大最小经纬度组成的矩形，然后通过100米（固定长度）分割后，如果最后一块大于25米，则算入一块新地块，如不满25米，则丢弃，根据此得到n块正方形，对这些正方形取中心点，就得到了n个地块采样点。

在讨论该需求时，大家都侧重于直接产生的思路，根据米数去计算偏移多少经纬度，然后再加起来，期间我也被直接产生的思路蒙蔽了眼睛，没有考虑到问题的复杂性：通过几十米去计算偏移量，想让做到精确恐怕是个天方夜谭了。

## 解决方案

我在走了一下午弯路后，发现这个浮于表面轻而易举得来的思路并不靠谱，大概就是说着简单、想着简单，做起来真要命。于是，我重新画了一下地块的分割及获取中心点的草图，一开始画数轴，突然有了思路：我在获取最大最小经度的米数差值时，已经知道了经度上有多少块地块分割，然后我把经度度数的差值按经度所占地块数量等额平分，再获取每块小地块的俩边边界点相加后除2，则得到这块小地块的经度中间点。纬度亦如此。然后经纬度再循环重新匹配，则得到整个大地块的所有中间点集合。

## 具体实现

其实重要的并不是具体实现，而是思考问题的思路，不要让自己或者别人一开始的思路带偏而一路走上黑，在陷入沼泽地的时候，时刻要思变通，变则通，通则久。

- 具体实现类

```java
mport java.util.*;
import java.util.stream.Collectors;

public class Main {


    /**
     * 通过经纬度点集合获取中心点
     * @param lngs 经度点集合
     * @param lats 纬度点集合
     * @return
     */
    public static List<Map<String,String>> getMidPoints(List<String> lngs, List<String> lats){
        //经纬度最大最小值
        double lngMax = Collections.max(lngs.stream().map(Double::parseDouble).collect(Collectors.toList()));
        double lngMin = Collections.min(lngs.stream().map(Double::parseDouble).collect(Collectors.toList()));
        double latMax = Collections.max(lats.stream().map(Double::parseDouble).collect(Collectors.toList()));
        double latMin = Collections.min(lats.stream().map(Double::parseDouble).collect(Collectors.toList()));
        //经度距离/米
        double lngMetre = LocationUtil.getDistance(latMax,lngMin,latMax,lngMax);
        //纬度距离/米
        double latMetre = LocationUtil.getDistance(latMin,lngMax,latMax,lngMax);
        int lngPart = (int) lngMetre/100;
        int latPart = (int) latMetre/100;
        int lngOver = (int) lngMetre%100;//
        int latOver = (int) latMetre%100;

        //经度计算
        double lngValue = lngMax-lngMin;
        if (lngOver > 25){
            lngPart++;
        }
        //每小块地间隔
        double lngBlock = lngValue/lngPart;
        double[] lngPoint = new double[lngPart+1];
        lngPoint[0] = lngMin;
        for (int i=1;i<lngPart;i++){
            lngMin += lngBlock;
            lngPoint[i] = lngMin;
        }
        lngPoint[lngPart] = lngMax;
        //经度中心点
        double[] lngMidPoints = new double[lngPart];
        for (int i = 0; i < lngPart; i++) {
            double midPoint = (lngPoint[i]+lngPoint[i+1])/2;
            lngMidPoints[i] = midPoint;
        }

        //纬度计算
        double latValue = latMax-latMin;
        if (latOver > 25){
            latPart++;
        }
        //每小块地间隔
        double latBlock = latValue/latPart;
        double[] latPoint = new double[latPart+1];
        latPoint[0] = latMin;
        for (int i=1;i<latPart;i++){
            latMin += latBlock;
            latPoint[i] = latMin;
        }
        latPoint[latPart] = latMax;
        //经度中心点
        double[] latMidPoints = new double[latPart];
        for (int i = 0; i < latPart; i++) {
            double midPoint = (latPoint[i]+latPoint[i+1])/2;
            latMidPoints[i] = midPoint;
        }

        List<Map<String,String>> list = new ArrayList<>();
        for (int j = 0; j < latMidPoints.length; j++) {
            for (int i = 0; i < lngMidPoints.length; i++) {
                Map<String,String> map = new HashMap<>();
                map.put("lat",latMidPoints[j]+"");
                map.put("lng",lngMidPoints[i]+"");
                list.add(map);
            }
        }
        return list;
    }


    public static void main(String[] args) {
        List<String> lngs = pointLng();
        List<String> lats = pointLat();
        getMidPoints(lngs,lats);


    }

    private static List<String> pointLng(){
        List<String> list = new ArrayList<>();
        list.add("116.512847");
        list.add("116.516636");
        list.add("116.516766");
        list.add("116.515431");
        list.add("116.512417");
        list.add("116.515302");
        list.add("116.514612");
        return list;
    }

    private static List<String> pointLat(){
        List<String> list = new ArrayList<>();
        list.add("35.755634");
        list.add("35.755645");
        list.add("35.751224");
        list.add("35.751255");
        list.add("35.752819");
        list.add("35.752543");
        list.add("35.753231");
        return list;
    }

}
```

- 通过俩点经纬度获取直线距离

```java
public class LocationUtil {
    private static double EARTH_RADIUS = 6378.137;

    private static double rad(double d) {
        return d * Math.PI / 180.0;
    }

    /**
     * 通过经纬度获取距离(单位：米)	 * 	 * @param lat1	 * @param lng1	 * @param lat2	 * @param lng2	 * @return 距离
     */
    public static double getDistance(double lat1, double lng1, double lat2, double lng2) {
        double radLat1 = rad(lat1);
        double radLat2 = rad(lat2);
        double a = radLat1 - radLat2;
        double b = rad(lng1) - rad(lng2);
        double s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2) + Math.cos(radLat1) * Math.cos(radLat2) * Math.pow(Math.sin(b / 2), 2)));
        s = s * EARTH_RADIUS;
        s = Math.round(s * 10000d) / 10000d;
        s = s * 1000;
        return s;
    }

    public static void main(String[] args) {
        double distance = getDistance(34.2675560000, 108.9534750000, 34.2464320000, 108.9534750000);
        System.out.println("距离" + distance / 1000 + "公里");
    }
}
```

