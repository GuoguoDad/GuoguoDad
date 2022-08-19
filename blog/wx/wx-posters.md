 <center> <h1>5分钟实现微信小程序海报页</h1></center>

##  备受青睐
- 不用安装。微信小程序是一种不需要安装即可使用的应用
- 跨平台性。一份代码即可多个平台使用
- 推广容易。用户扫一扫或搜一下即可打开应用

##  推荐使用
随着业务需求的不多发展，小程序开发必不可少，而经常遇到需要用canvas画海报，如何我们裸用canvas的api去画海报，在可读性、后期维护性方面比较差，给大家推荐一个非常好用画小程序海报的库，让canvas的api下沉下去，开发者不用关心api，专心于业务即可。

##  案例效果
<img src="https://img-blog.csdnimg.cn/34ff4c9f59514f6aa0578cea14668b66.png#pic_center" width="400" />

## 安装依赖
```javascript
1、npm i wx-canvas-2d -S

2、微信小程序开发者工具构建一下npm (工具 -> 构建 npm)
```

## 画海报
```javascript
import {
  WxCanvas2d,
  Text,
  Image,
  Debugger
} from 'wx-canvas-2d'

WxCanvas2d.use(Debugger)
    const canvas = new WxCanvas2d()
    canvas.create({
      query: '.poster-canvas', // 必传，canvas元素的查询条件
      rootWidth: 750, // 参考设备宽度 (即开发时UI设计稿的宽度，默认375，可改为750)
      bgColor: '#fff', // 背景色，默认透明
      component: this, // 自定义组件内需要传 this
      radius: 16 // 海报图圆角，如果不需要可不填
    })
    canvas.draw({
      series: [{
          type: Image,
          url: '../../images/bg.png',
          x: 0,
          y: 0,
          width: 600,
          height: 900,
          mode: 'scaleToFill',
          radius: 0,
          zIndex: 0
        },
        {
          type: Text,
          text: '标题最多最长十个字符',
          x: 40,
          y: 120,
          color: '#FFFFFF',
          fontSize: 52,
          fontWeight: 'bold'
        },
        {
          type: Text,
          text: '2022.06.17~2022.08.30',
          x: 120,
          y: 230,
          color: '#FBF1CD',
          fontSize: 32,
          fontWeight: 'bold'
        },
        {
          type: Text,
          text: '南京xxxx孝陵卫店南京xxxx孝陵卫店',
          width: 320,
          x: 50,
          y: 680,
          color: '#222222',
          fontSize: 28,
          lineHeight: 42
        },
        {
          type: Image,
          url: '../../images/sun.png',
          x: 400,
          y: 680,
          width: 160,
          height: 160,
          mode: 'scaleToFill',
          radius: 0,
          zIndex: 1
        },
      ]
    })
```

## 总结
- 提升开发效率，可维护性搞
- 思想非常值得参考，canvas api封装在库里，开发海报只需要配置，计算坐标、大小、位置即可


## 结束语
- 👀 目前专注于前端
- ⚙️ 在react、react-native开发方面有丰富的经验
- 🔭 最近在学习安卓，有自己的开源安卓项目，集成react-native热更新功能
- 我❤️ 思考、学习、编码和健身
- 如果文章对您有帮助，三连支持一下～O(∩_∩)O谢谢！

	
	

