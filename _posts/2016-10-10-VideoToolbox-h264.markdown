---
layout:     post
title:      "VideoToolbox h264编码"
subtitle:   ""
date:       2016-10-10 14:40:00
author:     "Ryan"
header-img: "img/home-bg-o.jpg"
tags:
    - webrtc
    - 音视频
    - iOS开发
---

##### 创建session
	VTCompressionSessionRef compressSession;
	OSStatus status = VTCompressionSessionCreate(NULL, 480, 640, kCMVideoCodecType_H264, NULL, NULL, NULL, 
	VideoCompressonOutputCallback, (__bridge void *)self ,&compressSession);
	
##### 用VTSessionSetProperty设置参数

* kVTCompressionPropertyKey_RealTime
// 实时编码输出，降低编码延迟
* kVTCompressionPropertyKey_ProfileLevel kVTProfileLevel_H264_Baseline_AutoLevel
// h264 profile, 直播一般使用baseline，可减少由于b帧带来的延时
* kVTCompressionPropertyKey_MaxKeyFrameInterval
// 关键帧间隔，GOPsize
* kVTCompressionPropertyKey_ExpectedFrameRate
// 期望帧率
* kVTCompressionPropertyKey_DataRateLimits
// 码率(比特率),上限,单位是bps
* kVTCompressionPropertyKey_AverageBitRate
// 码率,均值,单位是byte

##### 开始编码			
		VTCompressionSessionPrepareToEncodeFrames(compressSession);

##### 传入编码帧
	frameCount++;
	// 帧时间,如果不设置会导致时间轴过长
	CMTime presentationTimeStamp = CMTimeMake(frameCount,(int32_t)videoFrameRate);
	VTEncodeInfoFlags flags;
	CMTime duration = CMTimeMake(1, (int32_t)videoFrameRate);

	NSDictionary *properties = nil;
	if (frameCount % (int32_t)videoMaxKeyframeInterval == 0) {
	properties = @{(__bridge NSString *)kVTEncodeFrameOptionKey_ForceKeyFrame: @YES};}

	VTCompressionSessionEncodeFrame(compressionSession, pixelBuffer, presentationTimeStamp, duration, (__bridge CFDictionaryRef)properties, NULL, &flags);

##### 编码回调，每当系统编码完一帧之后，会异步掉用该方法
	static void VideoCompressonOutputCallback(void * VTref,void * CM_NULLABLE VTFrameRef,OSStatus status,VTEncodeInfoFlags infoFlags,CM_NULLABLE CMSampleBufferRef sampleBuffer) {}
	
##### 关键帧过去SPS和PPS,保存在h264文件开头即可

![Mou icon](https://raw.githubusercontent.com/dnqs123/dnqs123.github.io/master/postimage/post-img-VideoToolbox-h264-01)
	
##### 写入数据
![Mou icon](https://raw.githubusercontent.com/dnqs123/dnqs123.github.io/master/postimage/post-img-VideoToolbox-h264-02)