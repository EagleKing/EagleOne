---
layout: post
title: AVFoundtion 相关总结
description: 二维码扫描
image: assets/images/pic05.jpg
---

- ***AVCaptureMetadataOutput***这个类的 ***rectOfInterest*** 属性是基于横屏的坐标(**横屏坐标原点也在左上方,但是宽高互换,横纵坐标互换**)

- 初始化视频捕获

~~~ObjC
//获取摄像设备
AVCaptureDevice * device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
//创建输入流
AVCaptureDeviceInput * input = [AVCaptureDeviceInput deviceInputWithDevice:device error:nil];
if (!input) return;
//创建输出流
AVCaptureMetadataOutput * output = [[AVCaptureMetadataOutput alloc]init];

//设置代理
dispatch_queue_t dispatchSerialQueue = dispatch_queue_create("OutputQueue", DISPATCH_QUEUE_SERIAL);
 [output setMetadataObjectsDelegate:self queue:dispatchSerialQueue];

//设置有效扫描区域
CGRect scanCrop=[self getScanCrop:_scanWindow.bounds readerViewBounds:self.view.frame];
output.rectOfInterest = scanCrop;
//初始化链接对象
AVCaptureSession * session = [[AVCaptureSession alloc]init];
//高质量采集率
[session setSessionPreset:AVCaptureSessionPreset1920x1080];
[session addInput:input];
[session addOutput:output];

//设置扫码支持的编码格式(如下设置条形码和二维码兼容),EAN码符号有标准版（EAN-13）和缩短版（EAN-8）两种。标准版表示13位数字，又称为EAN13码，缩短版表示8位数字，又称EAN8
output.metadataObjectTypes=@[AVMetadataObjectTypeQRCode,//QR Code
                                                 AVMetadataObjectTypeEAN13Code,//EAN-13
                                                  AVMetadataObjectTypeEAN8Code,//EAN-8
                                                  AVMetadataObjectTypeCode128Code];//Code-128
    
//用于展示被输入设备捕获的视频
AVCaptureVideoPreviewLayer * layer = [AVCaptureVideoPreviewLayer layerWithSession:_session];
layer.videoGravity=AVLayerVideoGravityResizeAspectFill;
layer.frame=self.view.layer.bounds;
[self.view.layer insertSublayer:layer atIndex:0];
//开始捕获
[session startRunning];

~~~

-  采集到的数据通过***AVCaptureMetadataOutputObjectsDelegate*** 协议里方法

~~~ObjC
- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection
~~~
来获取.

- 从相册中读取二维码

~~~ObjC
//初始化一个监测器
CIDetector*detector = [CIDetector detectorOfType:CIDetectorTypeQRCode context:nil options:@{ CIDetectorAccuracy : CIDetectorAccuracyHigh }];
//读取图片里的 features
NSArray *features = [detector featuresInImage:[CIImage imageWithCGImage:image.CGImage]];
//读取第一个 feature
CIQRCodeFeature *feature = [features objectAtIndex:0];
//再读取各项所需要的属性
 NSString *scannedResult = feature.messageString;//扫描的 QRCode 的信息
......
~~~
- 打开手电筒

~~~ObjC
//获取设备
BOOL on = NO//默认是关闭手电筒的
AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
 if ([device hasTorch] && [device hasFlash])//先判断是否有闪光灯且是否有手电筒
 {  
            //在尝试配置设备的硬件相关属性之前，必须调用此方法。 当成功锁定设备以供您的代码配置时，此方法返回YES。 配置设备属性后，请调用unlockForConfiguration以释放配置锁定，并允许其他应用程序进行更改。
            [device lockForConfiguration:nil];
            if (on) {
                [device setTorchMode:AVCaptureTorchModeOn];
                [device setFlashMode:AVCaptureFlashModeOn];
                
            } else {
                [device setTorchMode:AVCaptureTorchModeOff];
                [device setFlashMode:AVCaptureFlashModeOff];
            }
            [device unlockForConfiguration];
}
~~~

