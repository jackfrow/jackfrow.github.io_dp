---
title: AVFoundation总结
date: 2019-9-9 10:42:49
tag: iOS
categories: 学习总结
---

#### 框架简介

> AVFoundation是iOS中用于处理音视频的框架，主要提供的功能有，视屏播放，视屏录制，以及视屏编辑等功能。

#### 概念介绍

##### AVAsset

> AVAsset是AVFoundation框架中抽象类，用来表示音频或者视屏资源,是一组音视频资源的类型,其中比较重要的属性有duration（持续时间）,tracks（资源轨道）。

```swift
  let asset = AVAsset(url: url)
```

##### AVPlayer

> An object that provides the interface to control the player’s transport behavior.

```swift
let player = AVPlayer(playerItem: <#T##AVPlayerItem?#>)//初始化
 player.seek(to: CMTime.zero) //跳转到播放位置
 player.play()//开始播放
 player.pause()//暂停
```

##### AVPlayerItem

> 该类主要是用于管理资源对象，提供播放数据源，旨在表示由AVPlayer播放的资产的呈现状态，并允许观察该状态，它控制着视频从创建到销毁的诸多状态。

```swift
let playItem = AVPlayerItem(asset: <#T##AVAsset#>)
```

##### AVPlayerLayer

> 用于显示视频内容，相当于大屏幕。里面有videoGravity，默认值 AVLayerVideoGravityResizeAspect.

##### AVAssetTrack

> AVAssetTrack是资源轨道，用于获取AVAsset中的资源，并加以处理

```swift
  //获取视屏资源中的视屏轨道(视屏交叉部分就会有多条轨道,这里只取第一条)
  guard let assetTrack = asset.tracks(withMediaType: .video).first else {continue }
```

多条轨道与显示的关系：

![DC9D6C74-F6D8-4D71-8456-7673C2F6353F](https://tva1.sinaimg.cn/large/006y8mN6gy1g6wo934s59j30kf05vjrq.jpg)

##### AVVideoCompositionInstruction

> 可以用作每段处理视屏的指令。

##### AVVideoCompositionLayerInstruction

> 用于显示界面的layer层,上图中可以看到1.2共存的部分,便会有多个LayerInstruction,就是一个资源中有多个轨道,并且轨道相交的部分就会产生多个LayerInstruction。

##### AVAssetExportSession

> 针对AVAsset源对象的内容进行转码，创建一个被指定输出形式的资源。

```swift
    // MARK: - 导出合成的视频
    func export(){
        let session = AVAssetExportSession.init(asset: compostion.copy() as! AVAsset, presetName: AVAssetExportPreset1920x1080)
        
        session?.outputURL = JRVideoEditor.createTemplateFileURL()
        session?.outputFileType = AVFileType.mp4
        session?.videoComposition = videoComposition
        
        session?.exportAsynchronously(completionHandler: {[weak self] in
            guard let strongSelf = self else {return}
            let status = session?.status
            if status == AVAssetExportSession.Status.completed {
                strongSelf.saveToAlbum(atURL: session!.outputURL!, complete: { (success) in
                    DispatchQueue.main.async {
                       strongSelf.showSaveResult(isSuccess: success)
                    }
                })
            }
        })
    }
```

> 

##### AVMutableComposition

> AVAsset的子类，可以用来新建视屏轨道，音频轨道，提取出资源中音频资源，视屏资源。

```swift
let composition = AVMutableComposition()
//新建一条视屏轨道
 guard let trackA = compostion.addMutableTrack(withMediaType: .video, preferredTrackID: trackId) else {return}
//提取视屏资源
let videoComposition = AVMutableVideoComposition(propertiesOf: composition)
```

![AVFoundation编辑结构图](https://tva1.sinaimg.cn/large/006y8mN6gy1g72g8fm2u4j30l50e3jw0.jpg)

#### 实战场景

##### 视屏播放

```swift
      guard let string = Bundle.main.path(forResource: "01_nebula", ofType: "mp4")else{
           return
        }
        let url = URL(fileURLWithPath: string)

        let asset = AVAsset(url: url)

        let playItem = AVPlayerItem(asset: asset)

         player = AVPlayer(playerItem: playItem)

        let layer = AVPlayerLayer(player: player)
        
        layer.frame = view.bounds
        
        view.layer.addSublayer(layer)
        
        player.play()
```

##### 视屏拼接

视屏拼接主要需要一条视频轨道用来存放视屏资源的数据。

```swift
 func buildVideoTrack()  {
        
        //使用invalid，系统会自动分配一个有效的trackId
        let trackId = kCMPersistentTrackID_Invalid
        
        guard let track = composition.addMutableTrack(withMediaType: .video, preferredTrackID: trackId) else {
            return
        }
        //视频片段插入时间轴时的起始点
        var cursorTime = CMTime.zero
        
        for asset in videos {
             //获取视频资源中的视频轨道
            guard let assetTrack = asset.tracks(withMediaType: .video).first else {
                continue
            }
            
            do {
                try track.insertTimeRange(CMTimeRange(start: .zero, duration: asset.duration), of: assetTrack, at: cursorTime)
                //光标移动到视频末尾处，以便插入下一段视频
                  cursorTime = CMTimeAdd(cursorTime, asset.duration)
            } catch  {
                print("insert error")
            }
        }
        
    }
```

##### 添加水印&添加动画

要想对视屏资源进行添加效果，主要是需要对AVMutableVideoComposition的animationTool属性进行处理。

> 水印代码

```swift

        // 1 - Set up the text layer
            let subtitle1Text = CATextLayer()
            subtitle1Text.font = "Helvetica-Bold" as CFTypeRef
            subtitle1Text.fontSize = 36
            subtitle1Text.frame = CGRect(x: 0, y: 0, width: size.width, height: 100)
            subtitle1Text.string = "jackfrow"
            subtitle1Text.alignmentMode = .center
            subtitle1Text.foregroundColor = UIColor.white.cgColor
        
            //2 - The usual overlay
            let overlayLayer = CALayer()
            overlayLayer.addSublayer(subtitle1Text)
            overlayLayer.frame = CGRect(x: 0, y: 0, width: size.width, height: size.height)
            overlayLayer.masksToBounds = true
            
            let parentLayer = CALayer()
            let videoLayer = CALayer()
            
            parentLayer.frame = CGRect(x: 0, y: 0, width: size.width, height: size.height)
            videoLayer.frame = CGRect(x: 0, y: 0, width: size.width, height: size.height)
            
            parentLayer.addSublayer(videoLayer)
            parentLayer.addSublayer(overlayLayer)
            
            // 3 - apply magic
            composition.animationTool = AVVideoCompositionCoreAnimationTool(postProcessingAsVideoLayer: videoLayer, in: parentLayer)
```

> 动画代码

```swift
        //1.overlay
              let animationImage = UIImage(named: "star.png")
              let overlayLayer1 = CALayer()
              overlayLayer1.contents = animationImage?.cgImage
              overlayLayer1.frame = CGRect(x: size.width/2 - 64, y: size.height/2 + 200, width:128, height: 128)
              overlayLayer1.masksToBounds = true
              
              let overlayLayer2 = CALayer()
              overlayLayer2.contents = animationImage?.cgImage
              overlayLayer2.frame = CGRect(x: size.width/2 - 64 , y: size.height/2 - 200, width: 128, height: 128)
              overlayLayer2.masksToBounds = true
        
        //2.3 - Twinkle
            let animationScale = CABasicAnimation(keyPath: "transform.scale")
                  animationScale.duration = 1.0
                  animationScale.repeatCount = 5
                  animationScale.autoreverses = true
                   //  // animate from half size to full size
            animationScale.fromValue = NSNumber(floatLiteral: 0.5)
            animationScale.toValue = NSNumber(floatLiteral: 1.0)
            animationScale.beginTime = AVCoreAnimationBeginTimeAtZero
              
            overlayLayer1.add(animationScale, forKey: "scale")
            overlayLayer2.add(animationScale, forKey: "scale")
            
            //3 - Composition
            let parentLayer = CALayer()
            let videoLayer = CALayer()
            parentLayer.frame = CGRect(x: 0, y: 0, width: size.width, height: size.height)
            videoLayer.frame = CGRect(x: 0, y: 0, width: size.width, height: size.height)
            parentLayer.addSublayer(videoLayer)
            parentLayer.addSublayer(overlayLayer1)
            parentLayer.addSublayer(overlayLayer2)
            
            composition.animationTool = AVVideoCompositionCoreAnimationTool(postProcessingAsVideoLayer: videoLayer, in: parentLayer)
```

##### 过渡效果

一段视屏资源播放出来主要是由它在视屏资源轨道中资源来决定的，效果类似于上图。

所以要想实现视屏的过渡效果，主要是需要多条视屏轨道交叉播放，然后对轨道交叉部分做一定的处理，就可以实现视屏的过渡效果。

```swift
///创建两条相交的视屏轨道  
func buildCompositionVideoTracks()  {
       //使用invalid，系统会自动分配一个有效的trackId
        let trackId = kCMPersistentTrackID_Invalid
        //创建AB两条视频轨道，视频片段交叉插入到轨道中，通过对两条轨道的叠加编辑各种效果。如0-5秒内，A轨道内容alpha逐渐到0，B轨道内容alpha逐渐到1
        guard let trackA = composition.addMutableTrack(withMediaType: .video, preferredTrackID: trackId) else {
            return
        }
        guard let trackB = composition.addMutableTrack(withMediaType: .video, preferredTrackID: trackId) else {
            return
        }
        
        let videoTracks = [trackA,trackB]
     //视频片段插入时间轴时的起始点
        var cursorTime = CMTime.zero
        //转场动画时间
        let transitionDuration = CMTime(value: 2, timescale: 1)
        for (index,value) in assets.enumerated() {
            //交叉循环A，B轨道
            let trackIndex = index % 2
            let currentTrack = videoTracks[trackIndex]
            //获取视频资源中的视频轨道
            guard let assetTrack = value.tracks(withMediaType: .video).first else {
                continue
            }
            do {
                //插入提取的视频轨道到 空白(编辑)轨道的指定位置中
                try currentTrack.insertTimeRange(CMTimeRange(start: .zero, duration: value.duration), of: assetTrack, at: cursorTime)
                //光标移动到视频末尾处，以便插入下一段视频
                cursorTime = CMTimeAdd(cursorTime, value.duration)
                //光标回退转场动画时长的距离，这一段前后视频重叠部分组合成转场动画
                cursorTime = CMTimeSubtract(cursorTime, transitionDuration)
            } catch {
                
            }
        }
        
    }
    /// 设置转场动画
    func setupTransition(for instruction: AVMutableVideoCompositionInstruction, fromLayer: AVMutableVideoCompositionLayerInstruction, toLayer: AVMutableVideoCompositionLayerInstruction ,type: TransitionType) {
        let identityTransform = CGAffineTransform.identity
        let timeRange = instruction.timeRange
        let videoWidth = self.videoComposition.renderSize.width
        if type == TransitionType.Push{
            let fromEndTranform = CGAffineTransform(translationX: -videoWidth, y: 0)
            let toStartTranform = CGAffineTransform(translationX: videoWidth, y: 0)
            
            fromLayer.setTransformRamp(fromStart: identityTransform, toEnd: fromEndTranform, timeRange: timeRange)
            toLayer.setTransformRamp(fromStart: toStartTranform, toEnd: identityTransform, timeRange: timeRange)
        }else {
            fromLayer.setOpacityRamp(fromStartOpacity: 1.0, toEndOpacity: 0.0, timeRange: timeRange)
        }
        
        //重新赋值
        instruction.layerInstructions = [fromLayer,toLayer]
    }

```

##### Demo

[AVFoundationDemo](https://github.com/jackfrow/AVFoundationDemo)

#### 参考链接

- [AVFoundation视屏开发总结](http://zichao.me/2016/03/28/视频开发总结/)
- [LearningAVFoundation之视频合成+转场过渡动画](https://juejin.im/post/5bee688ae51d45313b1ac683)
- [**在 iOS 上捕获视频**](https://objccn.io/issue-23-1/)
- [How to Play, Record, and Merge Videos in iOS and Swift](https://www.raywenderlich.com/5135-how-to-play-record-and-merge-videos-in-ios-and-swift)
- [AVFoundation Tutorial: Adding Overlays and Animations to Videos](https://www.raywenderlich.com/2734-avfoundation-tutorial-adding-overlays-and-animations-to-videos)
- [AVFoundation Programming Guide](https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40010188)