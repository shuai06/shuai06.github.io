# 弹出地图并缩放到指定经纬度


  

### 点击【地图查看】，弹出地图，并缩放到当前某一xx的范围
  
效果大致是这样(图没了)：

![](http://pqpog12fm.bkt.clouddn.com/%E5%9C%B0%E5%9B%BE%E6%9F%A5%E7%9C%8B.png)

![](http://pqpog12fm.bkt.clouddn.com/%E5%9C%B0%E5%9B%BE%E6%9F%A5%E7%9C%8B2.png)

  

------

1. 先把弹窗的页面元素写出来

```html
<!--弹出层时背景层DIV-->
<div id="fade" class="black_overlay"> </div>
<!-- 主体部分 -->
<div id="MyDiv" class="white_content">
	<div style="text-align: left; cursor: pointer; height: 40px; display:inline-block; float:left">
		<span style="font-size: 16px; font-weight:bold; line-height:40px;" id="dtval"></span>
	</div>
	<div style="float:right; text-align: right; cursor: pointer; height: 40px; display:inline-block">
		<button type="button" class="close" style="font-size: 30px;font-weight: bold; background: none;border: none;outline: none;position: relative;right:6px;top:4px " 
		onclick="CloseDiv('MyDiv','fade')" title="关闭详情">×</button>
	</div>
	<iframe frameborder="0" id="iframepagemap" name="iframepage" scrolling="no" style="width: 100%;height: 100%;" src=""></iframe>
</div>
```





2. 绑定按钮的点击事件

```html
<button  type="button"  class="btn btn-primary bottomBtn catMapBtn" onclick="ShowDiv('MyDiv','fade')">地图查看</button>     
//两个参数 分别是弹出主体和遮罩背景 的id
```

3. js文件中： 弹窗方法

```javascript
var initmap = false;

//弹出隐藏层
function ShowDiv(show_div,bg_div){
	document.getElementById(show_div).style.display='block';
	document.getElementById(bg_div).style.display='block' ;
	var bgdiv = document.getElementById(bg_div);
	bgdiv.style.width = document.body.scrollWidth;
	// bgdiv.style.height = $(document).height();
	$("#"+bg_div).height($(document).height());
	var iframe = document.getElementById("iframepagemap");
	if(!initmap){
		$("#iframepagemap").attr("src","../map/map.jsp");
		var i = 0;
        var finishmap = setInterval(function () {
            if (iframe.contentWindow.map!=null){
            	map = iframe.contentWindow.map;
                initM= iframe.contentWindow.initM;
                OpenLayers = iframe.contentWindow.OpenLayers;
                LayerSwitcherExt = iframe.contentWindow.LayerSwitcherExt;
                clearInterval(finishmap);
                initmap = true;
                // 项目范围ajax
            	LonatDetail(xmNameStr);
            }else{
                i++;
            }
        },100)
	}else{
		 // 项目范围ajax
    	LonatDetail(xmNameStr);
	}
};

```

4. 获取项目区域的ajax

```javascript

function LonatDetail(xmNameStr){
	$.ajax({
		url:baseurl + "Program.action",   
		type:"POST",
		data:{
			method:"progremLonat", 
			paramts:xmNameStr,
		},
		dataType: "json",
		success: function(data){
			addPolygon(data[0]['LONLAT']);
		},
		error: function(msg){
			alert("ajax Error!");
		}
	})
}

// 从后台拿到的data为：
/*
0: {LONLAT: "[{"x":100.12341032715,"y":30.675432620386},{"x":12…96102,"y":55.555}]"}
*/
```

![](http://pqpog12fm.bkt.clouddn.com/%E5%9C%B0%E5%9B%BE%E7%BC%A9%E6%94%BEdata%E6%A0%BC%E5%BC%8F.png)



5. 地图范围的缩放与绘制面

```javascript
// 弹出地图缩放至
function addPolygon(data){
    data2 = eval("("+data+")");   //解析json字符串为json对象	
    var paths = [];   // 存储坐标
    for (var a = 0; a < data2.length; a++) {
        var pf = new OpenLayers.Geometry.Point(
            data2[a]["x"],
            data2[a]["y"]);
        // 点的数组
        paths.push(pf);
    }
    var stylePolyline = {   //画面的样式
        strokeWidth : 2,
        strokeOpacity : 0.8,
        strokeColor : "blue",
        fillColor : "white",
        fillOpacity : 0.5
    };
    // 点组织成线
    var line = new OpenLayers.Geometry.LinearRing(paths);
    // 由线组织成面
    var polygon = new OpenLayers.Geometry.Polygon(line);
    var featureLine = new OpenLayers.Feature.Vector();
    featureLine.geometry = polygon;
    featureLine.style = stylePolyline;
    map.setLayerIndex(map.getLayer("drawLayer"), map.layers.length - 1);
    map.getLayer("drawLayer").removeAllFeatures();
    map.getLayer("drawLayer").addFeatures(featureLine);
    lonlat = featureLine.geometry.getBounds().getCenterLonLat();
    var zoom = map.getZoomForExtent(featureLine.geometry.getBounds()) - 1;
    if(zoom == 20){
        zoom = 19;
    }
    map.setCenter(lonlat,zoom);   //缩放到lonlat,  zoom是范围大小 

};


```



6. 关闭弹窗的方法

```javascript
//关闭弹出层
function CloseDiv(show_div,bg_div)
{
	document.getElementById(show_div).style.display='none';
	document.getElementById(bg_div).style.display='none';
};
```





---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E5%9C%B0%E5%9B%BE%E7%BC%A9%E6%94%BE%E5%88%B0%E6%8C%87%E5%AE%9A%E7%BB%8F%E7%BA%AC%E5%BA%A6/  

