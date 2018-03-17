---
layout:     post
title:      "iOS CAEmitterLayer(粒子动画)"
subtitle:   ""
date:       2016-8-20
author:     "xsnailcoder"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - iOS

    
---
## iOS CAEmitterLayer(粒子动画)

### 粒子动画的组成

#### **CAEmitterLayer** 粒子发射器的属性(具体看示例代码)

* **emitterCells**  粒子单元数组，可以创建多个粒子单元负责不同效果
* **emitterPosition** 发射的位置
* **emitterShape** 发射的形状
   * kCAEmitterLayerPoint //点
   * kCAEmitterLayerLine  //线
   * kCAEmitterLayerRectangle//矩形
   * kCAEmitterLayerCuboid //立方体
   * kCAEmitterLayerCircle //圆形
   * kCAEmitterLayerSphere //球形

* **emitterMode** 发射的模式
  * kCAEmitterLayerPoints //从发射器中 
  * kCAEmitterLayerOutline //从发射器边缘
  * kCAEmitterLayerSurface //从发射器表面
  * kCAEmitterLayerVolume //从发射器中点
* **renderMode** 渲染模式
  * kCAEmitterLayerUnordered //粒子是无序出现的，多个发射源将混合  
  * kCAEmitterLayerOldestFirst //声明久的粒子会被渲染在最上层
  * kCAEmitterLayerOldestLast //年轻的粒子会被渲染在最上层
  * kCAEmitterLayerBackToFront //粒子的渲染按照Z轴的前后顺序进行
  * kCAEmitterLayerAdditive //进行粒子混合





#### **CAEmitterCell**  粒子单元(具体看示例代码)

* **emissionLongitude** //设置粒子方向
* **birthRate** //设置粒子每秒弹出的个数
* **lifetime** //设置粒子的存活时间
* **scale**  //粒子缩放比例

**代码示例**


        //1.创建发射器
        let emitter = CAEmitterLayer()
        emitter.emitterSize = CGSize(width: 200, height: 100)
        //2.设置发射器的位置
        emitter.emitterPosition = CGPoint(x: self.view.bounds.width * 0.5 , y: self.view.bounds.height * 0.8)
        
        //2.1 设置发射器的形状
        emitter.emitterShape = kCAEmitterLayerSphere
        //2.2 设置发射模式
        emitter.emitterMode = kCAEmitterLayerSurface;
        //2.3 渲染模式
        emitter.renderMode = kCAEmitterLayerOldestFirst;
        emitter.masksToBounds = false;
        
        //3.开启三维效果
        emitter.preservesDepth = true
        
        //4.创建粒子、并且设置粒子的相关属性
        let cell = CAEmitterCell()
        
        //4.2设置粒子发射速度
        cell.velocity = 150
        cell.velocityRange = 100
        
        //4.3设置粒子的大小
        cell.scale = 0.3
        cell.scaleRange = 0.1
    
        //4.4设置粒子方向
        cell.emissionLongitude = CGFloat(-M_PI/2)
        cell.emissionRange = CGFloat(M_PI/4)
        
        //4.5设置粒子的存活时间
        cell.lifetime = 5
        cell.lifetimeRange = 3
        
        //4.6设置粒子每秒弹出的个数
        cell.birthRate = 200
        
        //4.7设置粒子的旋转
        cell.spin = CGFloat(M_PI_2)
        cell.spinRange = CGFloat(M_PI_2/4)
        
        //4.8设置粒子透明度
        cell.alphaSpeed = -0.2;
        cell.alphaRange = 0.5;
        
        //4.9 粒子内容的颜色
        cell.color = UIColor.white.cgColor
        //设置了颜色变化范围后每次产生的粒子的颜色都是随机的
        cell.redRange = 0.5;
        cell.blueRange = 0.5;
        cell.greenRange = 0.5;
        
        //4.10设置粒子展示的图片
        cell.contents = UIImage(named: "heart")?.cgImage
        
        //5.将粒子放到发射器中
        emitter.emitterCells = [cell]
 
        //6.将发射器的layer添加到父类的layer中
        view.layer.addSublayer(emitter)