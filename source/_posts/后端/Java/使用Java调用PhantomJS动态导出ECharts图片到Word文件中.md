---
title: 使用Java调用PhantomJS动态导出ECharts图片到Word文件中
date: 2018-10-01 22:50:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/01/java-word-chart.jpg
categories: 后端
tags:
  - Java
---

## 前言

最近在一个项目中遇到导出为Word文件的问题，导出Word的功能很简单，但是导出Word文件中包含数据库动态查询的统计数据而生成的Echarts图片，且导出Word的时机又是在凌晨的服务器定时执行，所以不能通过客户端访问统计页面时再去生成。

服务端语言使用Java语言，最开始考虑使用[JFreeChart][1]来生成统计图片，但是JFreeChart生成的图片很丑，且和ECharts的统计图效果截然不同。所以最终抛弃了使用`JFreeChart`，而采用了在服务端使用Java调用[PhantomJS][2]的指令来导出Ehcarts图片。所以主要的技术方案选型如下：

- `poi-tl`，一个简单的基于`Word`模版生成`Word`的工具。
- `PhantomJS`，一个基于`webkit`内核的无头浏览器，可在服务端程序实现加载、操作页面等功能

## 使用poi-tl导出Word

### poi-tl介绍

使用Java导出Word通常采用的是[Apache POI][3]的库，但是使用POI来导出Word，会书写大量的段落、样式等细节代码，代码量巨大，而且不易于维护。通过[poi-tl][4]只需要制作导出的模版，服务端一行代码调用，传入模版路径和`Map`或者`Bean`即可生成Word模版，代码量大大降低，以后导出样式不满意的时候，只需要修改Word模版文件即可。

> **注意**：`poi-tl`只能生成`docx`文件，对word2007之前的`doc`文档则不支持。

### Maven引入

```xml
<dependency>
    <groupId>com.deepoove</groupId>
    <artifactId>poi-tl</artifactId>
    <version>1.0.0</version>
</dependency>
```

> **注**：该包带入了`POI3.16`，如果系统中本身有低于`3.15`版本的`POI`，需要排除掉，否则生成Word时会报错。

### demo示例

首先，制作一个用于测试的word模版，使用`poi-tl`的标记语法做如下标记，如下图所示：

![测试word模版][5]

然后，构造一个需要渲染的model JavaBean类，如果有多个Bean，貌似只能通过继承来复用属性，采用组合的方式是渲染不了的，代码如下：

```java
/**
 * BaseProp
 * @author blinkfox on 2017-06-28.
 */
public class BaseProp {
    
    protected String baseProp;
    
    /**
     * 构造方法.
     * @param baseProp 基础属性
     */
    public BaseProp() {
        super();
    }

    public String getBaseProp() {
        return baseProp;
    }

    public void setBaseProp(String baseProp) {
        this.baseProp = baseProp;
    }
    
}
```

```java
/**
 * 测试旅游信息的bean.
 * @author blinkfox on 2017-06-28.
 */
public class Travel extends BaseProp {

    private String title;

    private String smallTitle;

    private String startDate;

    private String endDate;

    private int count;

    private double money;

    private String place1;

    private String place2;

    private PictureRenderData pic;

    /**
     * 构造方法.
     */
    public Travel() {
        super();
    }

    /*getter和setter方法.*/

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getSmallTitle() {
        return smallTitle;
    }

    public void setSmallTitle(String smallTitle) {
        this.smallTitle = smallTitle;
    }

    public String getStartDate() {
        return startDate;
    }

    public void setStartDate(String startDate) {
        this.startDate = startDate;
    }

    public String getEndDate() {
        return endDate;
    }

    public void setEndDate(String endDate) {
        this.endDate = endDate;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public double getMoney() {
        return money;
    }

    public void setMoney(double money) {
        this.money = money;
    }

    public String getPlace1() {
        return place1;
    }

    public void setPlace1(String place1) {
        this.place1 = place1;
    }

    public String getPlace2() {
        return place2;
    }

    public void setPlace2(String place2) {
        this.place2 = place2;
    }

    public PictureRenderData getPic() {
        return pic;
    }

    public void setPic(PictureRenderData pic) {
        this.pic = pic;
    }

}
```

最后，是模拟调用示例：

```java
/**
 * poi-tl库的使用示例.
 * Created by blinkfox on 2017/6/27.
 */
public class PoitlTest {

    private static final Logger log = LoggerFactory.getLogger(PoitlTest.class);

    /** 项目资源路径. */
    private static final String PATH = "F:/poitl-test/web";

    /** word模板路径. */
    private static final String DOC_PATH = PATH + "/template/test/test.docx";

    /** 图片路径. */
    private static final String PIC_PATH = PATH + "/template/test/pic.png";

    /** 输出文件及路径. */
    private static final String OUTPUT_PATH = "G:/test/poitl_out_word.docx";

    /**
     * 构造Bean型的data数据.
     * @return map
     */
    private static Travel buildBeanData() {
        Travel travel = new Travel();
        travel.setTitle("我的旅游日记");
        travel.setSmallTitle("再写日记");
        travel.setStartDate("2017-01-01");
        travel.setEndDate("2017-06-28");
        travel.setCount(3);
        travel.setPlace1("九寨沟");
        travel.setPlace2("天涯海角");
        travel.setMoney(1872.52);
        travel.setPic(new PictureRenderData(600, 400, PIC_PATH));
        travel.setBaseProp("这是");
        return travel;
    }

    /**
     * main方法.
     * @param args 数组参数
     */
    public static void main(String[] args) throws IOException {
        XWPFTemplate template = XWPFTemplate.compile(DOC_PATH).render(buildBeanData());

        FileOutputStream out = new FileOutputStream(OUTPUT_PATH);
        template.write(out);
        out.flush();
        out.close();
        template.close();
        log.info("通过'poi-tl'导出word成功!");
    }

}
```

最后，在导出的文件夹中可查看生成的word文件，如下所示：

![生成的Word文件结果][6]

## Java调用PhantomJS导出Ehcarts图片

### PhantomJS介绍

`PhantomJS`是一个基于`webkit`内核的无头浏览器，即没有UI界面的一个浏览器，只是其内的点击、翻页等人为相关操作需要程序设计实现。`PhantomJS`提供JavaScript API接口，即通过编写js程序可以直接与`webkit`内核交互，在此之上可以结合Java语言等，通过java调用js等相关操作，从而解决了以前`c/c++`才能比较好的基于`webkit`开发优质采集器的限制。

### PhantomJS的安装配置

#### windows环境

如果是在windows环境下，则在官网下载解压到某个目录后，将其bin目录加入到`path`变量中即可。

#### Linux环境

如果是在`Linux`环境下，在官网下载解压后，同样需要将`PhantomJS`的`bin`目录加入到`path`环境变量中，参考的命令和配置如下：

```bash
# 编辑配置文件.
vi ~/.bashrc

# 将PhantomJS的bin目录加入到PATH环境变量中.
export PHANTOMJS_HOME=/home/blinkfox/Documents/phantomjs-2.1.1-linux-x86_64
export PATH=${PHANTOMJS_HOME}/bin:$PATH

# 退出vi编辑器，使用source命令让刚才的配置即时生效.
source ~/.bashrc

# 测试PhantomJS是否安装成功，如果打出了版本信息，即安装成功.
phantomjs -v
```

### demo示例

这个demo的需求是这样的，我们使用Java调用`PhantomJS`的指令来在服务端加载含`ECharts`统计的图`html`文件，然后调用`ECharts`的生成图片方法，将图片传输到Java后台最终实现保存图片到指定路径中。

首先，制作`ECharts`的html页面，示例页面如下代码如下：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>测试的ECharts数据统计图</title>
</head>
<body>
    <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
    <div id="main" style="width:560px; height:270px;"></div>

<script type="text/javascript" src="/js/lib/jquery/jquery-1.9.1.min.js"></script>
<script type="text/javascript" src="/js/lib/echarts/v3/echarts.min.js"></script>
<script type="text/javascript">

// 基于准备好的dom，初始化echarts实例
var myChart = echarts.init(document.getElementById('main'));

// 指定图表的配置项和数据
var option = {
    title: {
        text: 'ECharts 入门示例'
    },
    animation: false, // 关闭动画效果
    tooltip: {},
    legend: {
        data:['销量']
    },
    xAxis: {
        data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
    },
    yAxis: {},
    series: [{
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
    }]
};

// 使用刚指定的配置项和数据显示图表。
myChart.setOption(option);

/**
 * ajax传输图片信息.
 */
function postImage() {
    // 向后台发起请求保存图片到指定目录.
    $.ajax({
        type: 'POST',
        url: '/test/saveImage',
        data: {picInfo: myChart.getDataURL()},
        success: function() {
            console.log('通过post请求传输数据成功!');
        }
    });
}
</script>
</body>
</html>
```

然后，使用`Servlet`来写一个服务端代码，用来获取`Base64`的图片信息并在后端解析保存图片，`Servlet`代码如下：

```java
/**
 * 保存Echarts统计图片的Servlet.
 * @author blinkfox on 2017-06-28.
 */
public class SaveImageServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    private static final Logger log = LoggerFactory.getLogger(SaveImageServlet.class);

    /**
     * 执行获取echarts图片的post请求.
     * @param request req
     * @param response resp
     * @throws ServletException Servlet异常.
     * @throws IOException IO异常.
     */
    @Override
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 获取图片信息.
        String picInfo = request.getParameter("picInfo");
        if (StringUtils.isBlank(picInfo)) {
            log.error("picInfo为空,未从前台获取到base64图片信息!");
            return;
        }

        this.getAndsaveImage(picInfo, "G:/test/image1.png");
    }

    /**
     * 获取并保存图片到本地.
     * @param picInfo 图片信息
     * @param imagePath 图片保存的路径
     */
    private void getAndsaveImage(String picInfo, String imagePath) {
        // 传递过程中  "+" 变为了 " ".
        String newPicInfo = picInfo.replaceAll(" ", "+");
        String picPath = decodeBase64(newPicInfo, new File(imagePath));
        log.warn("从echarts中生成图片的的路径为:{}", picPath);
    }

    /**
     * 解析Base64位信息并输出到某个目录下面.
     * @param base64Info base64串
     * @param picPath 生成的文件路径
     * @return 文件地址
     */
    private String decodeBase64(String base64Info, File picPath) {
        if (StringUtils.isEmpty(base64Info)) {
            return null;
        }

        // 数据中：data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABI4AAAEsCAYAAAClh/jbAAA ...  在"base64,"之后的才是图片信息
        String[] arr = base64Info.split("base64,");

        // 将图片输出到系统某目录.
        OutputStream out = null;
        try {
            // 使用了Apache commons codec的包来解析Base64
            byte[] buffer = Base64.decodeBase64(arr[1]);
            out = new FileOutputStream(picPath);
            out.write(buffer);
        } catch (IOException e) {
            log.error("解析Base64图片信息并保存到某目录下出错!", e);
        } finally {
            IOUtils.closeQuietly(out);
        }

        return picPath;
    }

}
```

然后，是书写PhantomJS脚本`echarts_load.js`来加载和调用图片下载的代码：

```javascript
var system = require('system');
var page = require('webpage').create();

// 如果是windows,设置编码为gbk，防止中文乱码,Linux本身是UTF-8
var osName = system.os.name;
console.log('os name:' + osName);
if ('windows' === osName.toLowerCase()) {
    phantom.outputEncoding="gbk";
}

// 获取第二个参数(即请求地址url).
var url = system.args[1];
console.log('url:' + url);

// 显示控制台日志.
page.onConsoleMessage = function(msg, lineNum, sourceId) {
    console.log('CONSOLE: ' + msg + ' (from line #' + lineNum + ' in "' + sourceId + '")');
};

//打开给定url的页面.
var start = new Date().getTime();
page.open(url, function(status) {
    if (status == 'success') {
        console.log('echarts页面加载完成,加载耗时:' + (new Date().getTime() - start) + ' ms');

        // 由于echarts动画效果，延迟500毫秒确保图片渲染完毕再调用下载图片方法.
        setTimeout(function() {
            page.evaluate(function() {
                postImage();
                console.log("调用了echarts的下载图片功能.");
            });
        }, 500);
    } else {
        console.log("页面加载失败 Page failed to load!");
    }

    // 3秒后再关闭浏览器.
    setTimeout(function() {
        phantom.exit();
    }, 3000);
});
```

最后，是使用`Java`来调用`PhantomJS`的指令，代码如下：

```java
/**
 * HttpTest.
 * @author blinkfox on 2017-06-28.
 * @version 1.0
 */
public class HttpTest {

    private static final Logger log = LoggerFactory.getLogger(HttpTest.class);

    private static final String PHANTOM_PATH = "phantomjs";

    //这里我的test.js是保存在G盘下面的phantomjs目录
    private static final String TEST_JS = "G:/test/phantom/test.js ";

    public void String downloadImage(String url) throws IOException {
        String cmdStr = PHANTOM_PATH + TEST_JS + url;
        log.info("命令行字符串:{}", cmdStr);

        Runtime rt = Runtime.getRuntime();
        try {
            rt.exec(cmdStr);
        } catch (IOException e) {
            log.error("执行phantomjs的指令失败！请检查是否安装有PhantomJs的环境或配置path路径！PhantomJs详情参考这里:http://phantomjs.org", e);
        }
    }

    /**
     * main.
     * @param args args
     * @throws IOException IO异常
     */
    public static void main(String[] args) throws IOException {
        downloadImage("http://127.0.0.1:8080/test/echart_test/test_echarts.html");
    }

}
```

通过调用测试代码即可在指定目录生成`Echarts`的图片啦！

联系上面生成`Word`的功能，两个功能一结合即可动态导出`ECharts`图片到`Word`文件中。

  [1]: http://www.jfree.org/jfreechart/index.html
  [2]: http://phantomjs.org/
  [3]: https://poi.apache.org/
  [4]: https://github.com/Sayi/poi-tl
  [5]: https://statics.sh1a.qingstor.com/2018/10/01/test-word-template.jpg
  [6]: https://statics.sh1a.qingstor.com/2018/10/01/test-out-result.jpg
