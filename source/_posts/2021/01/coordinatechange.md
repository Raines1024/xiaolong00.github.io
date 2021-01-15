---
title: 地理坐标转换
date: 2021-01-15 15:58:39
tags: [编程, 过去]
category:
- 200 工作类
- 210 代码类
- 213 业务脚本
---

## 使用：Mysql连接封装脚本

## 地理坐标转换
### 通用地理位置计算工具类
- 已使用将火星坐标转wgs84坐标系，使用AbcMapToGpsExact方法，结果比较精确，会丢失精度。  
```
public class MapUtil {
    private static final Logger logger = LoggerFactory.getLogger(MapUtil.class);
    private static double x_pi = 3.14159265358979324 * 3000.0 / 180.0;

    /**
     * 计算两经纬度点之间的距离（单位：米）
     *
     * @param lng1 经度1
     * @param lat1 纬度1
     * @param lng2 经度2
     * @param lat2 纬度2
     * @return 距离，单位：米
     */
    public static double getDistance(double lng1, double lat1, double lng2, double lat2) {
        double radLat1 = Math.toRadians(lat1);
        double radLat2 = Math.toRadians(lat2);
        double a = radLat1 - radLat2;
        double b = Math.toRadians(lng1) - Math.toRadians(lng2);
        double s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2) + Math.cos(radLat1)
                * Math.cos(radLat2) * Math.pow(Math.sin(b / 2), 2)));
        s = s * 6378137.0;// 取WGS84标准参考椭球中的地球长半径(单位:m)
        s = Math.round(s * 10000) / 10000;
        return s;
    }

    /**
     * 把gps坐标转换成百度坐标 (BD-09)
     *
     * @param gpsLatLng
     * @return
     */
    public static LatLng gpsToBaiDu(LatLng gpsLatLng) {
        LatLng ggLatLng = GpsToAbcMap(gpsLatLng);
        return ggToBaiDu(ggLatLng);
    }

    /**
     * 把火星坐标转换成百度坐标 (BD-09)
     *
     * @param gpsLatLng
     * @return
     */
    public static LatLng ggToBaiDu(LatLng gpsLatLng) {
        LatLng bdLatLng = null;
        double huo_x = gpsLatLng.longitude;
        double huo_y = gpsLatLng.latitude;
        double z = Math.sqrt(huo_x * huo_x + huo_y * huo_y) + 0.00002 * Math.sin(huo_y * x_pi);
        double theta = Math.atan2(huo_y, huo_x) + 0.000003 * Math.cos(huo_x * x_pi);
        bdLatLng = new LatLng(z * Math.sin(theta) + 0.006, z * Math.cos(theta) + 0.0065);
        return bdLatLng;
    }

    /**
     * 将百度坐标转换为Gps坐标
     *
     * @param baiduLatLng
     * @return
     */
    public static LatLng baiduToGps(LatLng baiduLatLng) {
        LatLng ggLatLng = baiDuToGg(baiduLatLng);
        return AbcMapToGpsExact(ggLatLng);
    }

    /**
     * 将百度坐标 (BD-09)转换为火星坐标
     *
     * @param bdLatLng
     * @return
     */
    public static LatLng baiDuToGg(LatLng bdLatLng) {
        LatLng googleGps = null;
        double bd_x = bdLatLng.longitude - 0.0065;
        double bd_y = bdLatLng.latitude - 0.006;
        double z = Math.sqrt(bd_x * bd_x + bd_y * bd_y) - 0.00002 * Math.sin(bd_y * x_pi);
        double theta = Math.atan2(bd_y, bd_x) - 0.000003 * Math.cos(bd_x * x_pi);
        googleGps = new LatLng(z * Math.sin(theta), z * Math.cos(theta));
        return googleGps;
    }

    /**
     * 求出gps转高德坐标的偏移量
     *
     * @param latLng
     * @return
     */
    private static LatLng abcDelta(LatLng latLng) {
        LatLng reLatLng;
        if (outOfChina(latLng.latitude, latLng.longitude)) {
            reLatLng = new LatLng(0, 0);
            return reLatLng;
        }
        double dLat = transformLat(latLng.longitude - 105.0, latLng.latitude - 35.0);
        double dLon = transformLon(latLng.longitude - 105.0, latLng.latitude - 35.0);
        double radLat = latLng.latitude / 180.0 * pi;
        double magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        double sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * pi);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * pi);
        reLatLng = new LatLng(dLat, dLon);
        return reLatLng;

    }

    /**
     * 将高德坐标转换为gps坐标，结果比较精确
     *
     * @param abcLatLng
     * @return
     */
    public static LatLng AbcMapToGpsExact(LatLng abcLatLng) {
        double initDelta = 0.01;
        double threshold = 0.000000001;
        double dLat = initDelta, dLon = initDelta;
        double mLat = abcLatLng.latitude - dLat;
        double mLon = abcLatLng.longitude - dLon;
        double pLat = abcLatLng.latitude + dLat;
        double pLon = abcLatLng.longitude + dLon;
        double wgsLat, wgsLon, i = 0;
        while (true) {
            wgsLat = (mLat + pLat) / 2;
            wgsLon = (mLon + pLon) / 2;
            LatLng tmp = GpsToAbcMap(new LatLng(wgsLat, wgsLon));
            dLat = tmp.latitude - abcLatLng.latitude;
            dLon = tmp.longitude - abcLatLng.longitude;
            if ((Math.abs(dLat) < threshold) && (Math.abs(dLon) < threshold))
                break;

            if (dLat > 0) pLat = wgsLat;
            else mLat = wgsLat;
            if (dLon > 0) pLon = wgsLon;
            else mLon = wgsLon;

            if (++i > 10000) break;
        }
        return new LatLng(wgsLat, wgsLon);
    }

    /**
     * 采用模糊算法计算将高德坐标转换为gps坐标，结果不精确
     *
     * @param abcLatLng
     * @return
     */
    public static LatLng AbcMapToGps(LatLng abcLatLng) {
        LatLng gpsLatLng;
        LatLng delta = abcDelta(abcLatLng);
        gpsLatLng = new LatLng(abcLatLng.latitude - delta.latitude, abcLatLng.longitude - delta.longitude);
        return gpsLatLng;
    }

    /**
     * 将gps坐标转换为高德坐标
     *
     * @param gpsLatLng
     * @return
     */
    public static LatLng GpsToAbcMap(LatLng gpsLatLng) {
        LatLng abcLatLng = null;
        if (outOfChina(gpsLatLng.latitude, gpsLatLng.longitude)) {
            abcLatLng = new LatLng(gpsLatLng.latitude, gpsLatLng.longitude);
            return abcLatLng;
        }
        double dLat = transformLat(gpsLatLng.longitude - 105.0, gpsLatLng.latitude - 35.0);
        double dLon = transformLon(gpsLatLng.longitude - 105.0, gpsLatLng.latitude - 35.0);
        double radLat = gpsLatLng.latitude / 180.0 * pi;
        double magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        double sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * pi);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * pi);
        abcLatLng = new LatLng(gpsLatLng.latitude + dLat, gpsLatLng.longitude + dLon);
        return abcLatLng;
    }

    private final static double pi = 3.14159265358979324;
    private final static double a = 6378245.0;
    private final static double ee = 0.00669342162296594323;

    private static boolean outOfChina(double lat, double lon) {
        if (lon < 72.004 || lon > 137.8347)
            return true;
        if (lat < 0.8293 || lat > 55.8271)
            return true;
        return false;
    }

    private static double transformLat(double x, double y) {
        double ret = -100.0 + 2.0 * x + 3.0 * y + 0.2 * y * y + 0.1 * x * y + 0.2 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(y * pi) + 40.0 * Math.sin(y / 3.0 * pi)) * 2.0 / 3.0;
        ret += (160.0 * Math.sin(y / 12.0 * pi) + 320 * Math.sin(y * pi / 30.0)) * 2.0 / 3.0;
        return ret;
    }

    private static double transformLon(double x, double y) {
        double ret = 300.0 + x + 2.0 * y + 0.1 * x * x + 0.1 * x * y + 0.1 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(x * pi) + 40.0 * Math.sin(x / 3.0 * pi)) * 2.0 / 3.0;
        ret += (150.0 * Math.sin(x / 12.0 * pi) + 300.0 * Math.sin(x / 30.0 * pi)) * 2.0 / 3.0;
        return ret;
    }
}

```

### 经纬度坐标类
```
public class LatLng {
    private static final String a = LatLng.class.getSimpleName();
    public final double latitude;
    public final double longitude;

    /**
     *
     * @param lat latitude
     * @param lng longitude
     */
    public LatLng(double lat, double lng) {
        int var5 = (int) (lat * 1000000.0D);
        int var6 = (int) (lng * 1000000.0D);
        this.latitude = (double) var5 / 1000000.0D;
        this.longitude = (double) var6 / 1000000.0D;
    }

    public String toString() {
        String var1 = new String("latitude: ");
        var1 = var1 + this.latitude;
        var1 = var1 + ", longitude: ";
        var1 = var1 + this.longitude;
        return var1;
    }
}
```























