 
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
