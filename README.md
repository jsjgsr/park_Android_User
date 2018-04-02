 
智能车库
==== 


# 功能介绍
 1、用户的登录注册，修改密码<br>
 2、用ListView和SimpleAdapter展示用户历史停车记录，下拉刷新<br>
 3、二维码生成工具类<br>
 4、调用百度SDK显示指定区域，并添加地点标签。通过访问服务器，获取已存在的车库的x,y坐标并显示<br>
 5、存车操作，向服务器发送存车请求<br>
 
# 详细实现
### 1、二维码生成
    调用了谷歌com.google.zxing第三方包<br>
    主要函数：<br>
##### BitMatrix matrix = new MultiFormatWriter().encode(str, BarcodeFormat.QR_CODE, widthAndHeight, widthAndHeight);//// 生成矩阵
  调用zxing的MultiFormatWriter().encode()方法，生成一个矩阵BitMatrix。其中有两个重载方法，这个是没有设置hints的,所以默认用ISO-8859-1编码
  进入endcode函数。使用了获得一个Writer对象，最重要获得一个QRCode对象，使用Encoder.encode方法<br>
    
1、数据分析。在encode方法方法中，需要先将输入的要变成二维码的字符转换成ASCII码，根据ASCII码判断是数字，字母还是数字和字母的组合。即数据分析。每种调用的方法不一样。这里我们只是传入数字<br>


    static void appendNumericBytes(CharSequence content, BitArray bits) {
        int length = content.length();
        int i = 0;
        while(i < length) {
            int num1 = content.charAt(i) - 48;
            int num2;
            if(i + 2 < length) {
                num2 = content.charAt(i + 1) - 48;
                int num3 = content.charAt(i + 2) - 48;
                bits.appendBits(num1 * 100 + num2 * 10 + num3, 10);
                i += 3;
            } else if(i + 1 < length) {
                num2 = content.charAt(i + 1) - 48;
                bits.appendBits(num1 * 10 + num2, 7);
                i += 2;
            } else {
                bits.appendBits(num1, 4);
                ++i;
            }
        }
        }
2、数据编码，调用不同模式下的算法来吧原始数据进行编码，并转换为位流。bitarray<br>
3、然后把数据码序分段，变成块，以便构成纠错码子<br>
4、将各种信息（模式，版本信息，纠错信息）加入到QRcode对象中<br>
### 2、百度地图的调用
阅读文档去百度官方获得秘钥http://lbsyun.baidu.com/index.php?title=androidsdk/guide/create-project/ak
使用：<br>
1、	将第三方项目<br>
2、	往工程中添加jar文件<br>
3、	配置：<br>
 a)	在application中添加开发密钥<br>
 http://lbsyun.baidu.com/index.php?title=androidsdk/guide/create-project/hellomap
4、	使用在Layout中加入百度地图的view<br>

      <com.baidu.mapapi.map.MapView
          android:id="@+id/bmapView"
          android:layout_width="fill_parent"
          android:layout_height="fill_parent"
          android:clickable="true" />
5、	在对应的Activity获取Mapview。阅读源码我们知道百度地图的MapView是继承自viewgroup的。百度地图在画图的时候是将地图分为好多层，每一层画一个背景，例如最低下一次画成块的背景颜色，第二层画街道线条，第三层画房屋线条，第四层写对应的街道名称，第五层……..<br>
对于调用者来说，我们一方面可以修改百度给我们提供好的图层，例如换个道路线条的颜色等等，另一方面我们还可以在原有的图层基础上添加我们自己定义的图层
6、	有了图层以后我们就可以在地图上根据坐标表我们需要的坐标点。在这个项目的需求中，用户要连接服务器，获取车库信息，并将车库标注在地图上。我们要解决的第一个问题是用什么信息标，和如何在地图上显示。<br>
a)	首先第一个问题，如何标。地图是一个二维坐标来定位的，也就是经纬度，有了一个坐标的经度纬度，我们就可以在地图中准确的标注出来。下一步就是标注<br>
b)	百度地图提供了LatLng对象来表示坐标LatLng(double var1, double var3)，通过这个函数将值给LatLng对象，要注意var1是纬度,var3是经度，且出入的值需要是double类型。为什么要选用double来做经纬度呢？float类型或者其他类型不行吗？<br>
float与double的区别：float是单精度浮点，一共可以有8位有效数字。Double是双精度浮点，一共有17位有效数字。Double所表示的会更精确，但是要占空间大。
在float转double是不会出错，但是double转float时会出现意想不到的事情，因为double精度比float的高，我们知道浮点数是无法在计算机中准确表示的，例如0.1在计算机中只是表示成了一个近似值，因此，对浮点数的运算时结果具有不可预知性。<br>
Float转int前面要加int强转<br>
Int转float或者double都会加上小数点后面是0<br>
Double转float一样<br>
Float转double会出现精度错位，会出现好多位<br>
c)	坐标有了，我们还需要一个小图标来显示这个坐标的位置，我们用<br>
            BitmapDescriptor bitmap = BitmapDescriptorFactory<br>
                                .fromResource(R.drawable.icon_marka);<br>
   来获取图片。<br>
d)	将上面的是信息加入到一个对象中，利用建造者模式，同时还可以进行其他配置<br>
OverlayOptions option = new MarkerOptions()<br>
                    .position(point)<br>
                    		.icon(bitmap)<br>
.zIndex(5);<br>
		并将坐标显示出来<br>
Marker  marker = (Marker) (mBaiduMap.addOverlay(option));<br>
mBaiduMap是Layout中的那个view，使用addOverlay将之前配好的放到地图上。Marker对象还可以绑定数据，用于点击获取<br>
Bundle bundle = new Bundle();//制定坐标点的信息<br>
            bundle.putSerializable("info", map.getCkname());<br>
            bundle.putSerializable("mid",""+ mid);<br>
            marker.setExtraInfo(bundle);<br>
最后需要将整个视图定位到以一个点为中心位置就需要调用<br>
MapStatusUpdate msu = MapStatusUpdateFactory.newLatLng(point);<br>
        mBaiduMap.setMapStatus(msu);<br>
point是你希望的那个放在中间位置的点。<br>
7）坐标点画出来了，但是只能看，我需要的是点一下表示用户想要存这个车库，下面来说一下坐标的点击事件。在不同的Button中点击事件是要重新onclick方法，在百度地图中也是<br>
		控件mBaiduMap要设置setOnMarkerClickListener方法<br>
  
     mBaiduMap.setOnMarkerClickListener(new BaiduMap.OnMarkerClickListener() {
         @Override
         public boolean onMarkerClick(Marker marker) {
             final String info = (String)marker.getExtraInfo().get("info");
             final String mid = (String)marker.getExtraInfo().get("mid");
             //infowindow中的布局,显示小标签的样式
             TextView tv = new TextView(MapActivity.this);
             tv.setBackgroundResource(R.drawable.popup);
             tv.setPadding(20, 10, 20, 20);
             tv.setTextColor(Color.BLACK);
             tv.setText(info);//小标签的名字
             tv.setGravity(Gravity.CENTER);
             bitmapDescriptor = BitmapDescriptorFactory.fromView(tv);
当点击的时候可以获得marker对象中的数据，同时设置一个要弹出的窗口的TextView，当用户点击坐标，会有一个提示说这个坐标的车库的名字。	<br>
InfoWindow infoWindow = new InfoWindow(bitmapDescriptor, llInfo, -120, listener);//配置给InfoWindow。bitmapDescriptor是刚才设置的<br>TextView，llInfo是通过marker得到的LatLng对象，-120是相对于你点击的marker显示的位置，向上移动120，listener是你点击这个infoWindow时的回调监听
//infowindow点击事件，点击标签的监听事件<br>

                InfoWindow.OnInfoWindowClickListener listener = new InfoWindow.OnInfoWindowClickListener() {
                    @Override
                    public void onInfoWindowClick() {
                        //隐藏infowindow
                        Intent intent  = new Intent(MapActivity.this,MainActivity.class);
                        Bundle bundle = new Bundle();
                        bundle.putSerializable("mid", mid);
                        bundle.putSerializable("locName", info);
                        intent.putExtras(bundle);
                        startActivity(intent);
                        finish();
                        mBaiduMap.hideInfoWindow();
                    }
                };
点击之后就会跳转到下一个页面<br>
最后让infowindow显示：<br>
mBaiduMap.showInfoWindow(infoWindow);<br>

