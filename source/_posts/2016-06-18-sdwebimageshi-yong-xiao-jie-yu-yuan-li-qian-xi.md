---
layout: post
title: "SDWebImage使用小结与原理浅析"
date: 2016-06-18 14:32:22 +0800
comments: true
categories: iOS开发
keywords: iOS开发 Objective-C SDWebImage
description: 
---

本文主要介绍一下使用SD的正确方式，并浅析其实现原理。

在一般的SD使用中重要的有以下几个类：

``` Objective-C
SDWebImageDownloader
SDWebImageManager
SDImageCache
```
以及这两个类别：

```
UIButton+WebCache
UIImageView+WebCache
```
如果对更深一些的实现细节有兴趣，还可以看下
`SDWebImageDownloaderOperation`这个类，继承自`NSOperation`。`SDWebImageDecoder`是用来对图片进行解码操作的类，UIImage在显示到UI界面上时都会进行解码，SD的默认实现是先解码再返回UIImage给用户。详见下文。

<!--more-->

###1.`UIImageView+WebCache`的具体实现

先看代码，所有的实现最终都会调用这个方法：

```
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)completedBlock;
```
第一步先取消该UIView之前的下载队列:

```
[self sd_cancelCurrentImageLoad];
- (void)sd_cancelCurrentImageLoad {
    [self sd_cancelImageLoadOperationWithKey:@"UIImageViewImageLoad"];
}
```
![](/images/2016/06/11.png)

SD中有一个UIView的类别`UIView+WebCacheOperation`，给每个UIView用关联对象的方式添加了一个字典属性，用来记住此UIView的图片下载的Operation，UIView每次进行下载前先取消上一次的下载Operation，从而避免同一个UIView的并行下载。
`conformsToProtocol:`这个方法是用来判断消息接收者是否实现了某个协议。
第二步给当前要下载的图片URL设置一个key:

```
objc_setAssociatedObject(self, &imageURLKey, url, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
```
这一步一般我们用不着，主要是为了下方的扩展方法用：

```
- (NSURL *)sd_imageURL {
    return objc_getAssociatedObject(self, &imageURLKey);
}
```
接着往下，做了一层url为空的保护，如果为空则用block回调一个错误。
ActivityIndicator一般说来用不上，可以略过。
如果url不为空，那么进入下载图片的操作，在下载完成的block里进行回调和调用`setNeedsLayout`进行布局。
需要注意的是如果是单独下载图片，请使用 SDWebImageManager进行下载，不要直接使用`SDWebImageDownloader`，因为它没有做缓存，把图片缓存到内存和磁盘文件是`SDWebImageManager`在维护。

`UIButton+WebCache`里边的操作是类似的，只是多一个一个状态而已。

###2.开始分析图片下载过程

(1)先看下 `SDWebImageOptions`的枚举参数

`SDWebImageRetryFailed`
系统默认一张图片下载失败后，会把该图片加入失败的黑名单里，如果不传递该option，则下次下载同样的url时直接返回失败。如果传入该option，那么即使该图片之前下载失败了，也会再次进行下载：

![](/images/2016/06/12.png)


`SDWebImageLowPriority`，表示低优先级，例如当scrollView在减速时会延迟下载。

`SDWebImageHighPriority`，表示高优先级，会把该次下载操作移动到下载队列的最前面。

`SDWebImageCacheMemoryOnly`， 表示下载的图片只缓存到内存，不进行写文件，不做磁盘缓存。

`SDWebImageProgressiveDownload`， 表示渐进式的下载和显示，SD默认是把图片全部下载后再显示出来，但是传入该参数可以做到下载一部分图片后先显示这一部分，也就是边下载边显示，效果上就是通常的一个人体的照片，先看到头，再看到肩膀，最后看到腿。当然这种下载图片的方式需要服务器的支持才行。

`SDWebImageRefreshCached` 传入该option，则即使本地已经缓存了该图片，那么仍然会发起网络请求跟服务器通信，然后服务器的返回参数决定是否要再次下载该url表示的图片。同时缓存的方式改成使用NSURLCache，另如果图片命中缓存且远端图片同时有更新的话，会回调两次completionBLock。该参数主要是用于应对同一个url，但是图片会更改的情况。

SD的源码中主要是有以下几个影响：
命中缓存后先回调block，然后继续去下载图片，注意看这里在回调block之后并没有return，而是让代码往下走。

![](/images/2016/06/13.png)

同时如果图片命中NSURLCache的缓存，不进行回调。注意此处如果命中缓存则downloadImage肯定是空的。

![](/images/2016/06/14.png)

原理见下图以及[这篇博客](https://github.com/ChenYilong/ParseSourceCodeStudy/blob/master/02_Parse%E7%9A%84%E7%BD%91%E7%BB%9C%E7%BC%93%E5%AD%98%E4%B8%8E%E7%A6%BB%E7%BA%BF%E5%AD%98%E5%82%A8/iOS%E7%BD%91%E7%BB%9C%E7%BC%93%E5%AD%98%E6%89%AB%E7%9B%B2%E7%AF%87.md)。


![](/images/2016/06/15.png)


`SDWebImageContinueInBackground` 这个最简单了，字面意思就是APP进入后台后继续下载图片，只不过其代码实现略复杂，臣妾看的也不是太懂，故略过不表。

`SDWebImageHandleCookies` 缓存相关，HTTP的缓存不大懂，略过。

`SDWebImageAllowInvalidSSLCertificates` 跟图片下载权限和服务器认证相关，一般用不上该option，具体原理不明，略过。。。

`SDWebImageDelayPlaceholder` 这个选项一般也不用，如果设置的话，placeholder的UIImage就没效果了。

`SDWebImageTransformAnimatedImage` 字面上看跟gif动图相关？不懂略过。

`SDWebImageAvoidAutoSetImage` 这个选项没啥营养，就是给你一个机会在下载完成的时候可以对原图片做些操作，系统就不自动帮你设置了，而是直接回调block，见下图：

![](/images/2016/06/16.png)

(2)关于`SDWebImageDownloader`这个类，其实就是个实际去下载的类，它的那些Option跟`SDWebImageManager`里的Option是类似的，只不过`SDWebImageManager`是面向用户的，封装了查找缓存的一些功能，更友好一些而已。下图就是两个Option枚举之间的转换，其实意思上是差不多的：

![](/images/2016/06/17.png)


然后前面提到的同一个url所代表的图片可能会更新的问题也在这个类里有体现：

![](/images/2016/06/18.png)

这里的这个HTTP头的设置，跟下方的`SDWebImageDownloaderOperation`所拿到的response里的statusCode是有一定关系的：

![](/images/2016/06/19.png)

具体实现304 Not Modified的方案见上面的链接。


###3.再看下缓存部分`SDImageCache`，源码比较简单，提几个关键点即可。

①把图片的url当成key，然后转成一个唯一的字符串，来作为存储本地磁盘的文件名

![](/images/2016/06/20.png)

②所谓内存缓存，其实就是一个 NSCache，相当于可变字典，只不过多了cost参数而已

![](/images/2016/06/21.png)

③查找缓存的过程就是，先查内存里(NSCache)有没有，有就返回，没有就继续查磁盘文件有没有，有的话先放到内存里，然后返回。用一个单独的队列来进行磁盘读写。

![](/images/2016/06/22.png)

④缓存自动清理的时机
无论是APP退到后台，或者APP被直接杀掉，都会进行图片的清理，见下图

![](/images/2016/06/23.png)

该方法里会检查图片的有效期，默认是7天，如果过期则删除。
用到了 `NSURLContentModificationDateKey`这个key，表示文件的修改时间，多数情况下都是图片文件的创建时间，因为基本下载好了以后就不会去修改了。

另如果你设置了最大的图片存储空间，那么系统也会在同一时间点做检查并清理。即使未过期，也会清理一些，按照文件创建的时间来排序做清理，更早创建的优先被清理。比较有意思的就是，清理工作会持续到图片只占你设定值的一半。

![](/images/2016/06/24.png)

⑤关于图片的解码，从磁盘中读取时，以及从网络上下载完成图片时，都会先解码再返回

![](/images/2016/06/25.png)

![](/images/2016/06/26.png)

解码函数可调用多次，只有第一次有效，解码一次之后anyAlpha的值就为YES，所以解码函数调用多次也无太大影响

![](/images/2016/06/27.png)

另SD的对图片的回调有UIImage和NSData两个参数，也就是说，UIImage是解码之后的图片，但是格式为NSData的imageData却是下载下来的图片原始数据。但是由于作者对方法封装的原因，使用`SDWebImageManager`的回调确实没有imageData的:

```
typedef void(^SDWebImageCompletionWithFinishedBlock)(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL);
```

而使用`SDWebImageDownloader`的回调block是有NSData的参数回调的：

```
typedef void(^SDWebImageDownloaderCompletedBlock)(UIImage *image, NSData *data, NSError *error, BOOL finished);
```

SD存磁盘其实是拿图片的NSData的数据去存储的，并没有直接把解码的UIImage图片拿去存盘，所以在读磁盘文件时才需要先解码再返回了，代码见下方：

```
if (downloadedImage && finished) {
	[self.imageCache storeImage:downloadedImage recalculateFromImage:NO imageData:data forKey:key toDisk:cacheOnDisk];
}
```

注意此处的参数，中间的BOOL值是NO，所以在缓存的类`SDImageCache`里是这么处理的：

```
    if (toDisk) {
        dispatch_async(self.ioQueue, ^{
            NSData *data = imageData;

            if (image && (recalculate || !data)) {
#if TARGET_OS_IPHONE
                // We need to determine if the image is a PNG or a JPEG
                // PNGs are easier to detect because they have a unique signature (http://www.w3.org/TR/PNG-Structure.html)
                // The first eight bytes of a PNG file always contain the following (decimal) values:
                // 137 80 78 71 13 10 26 10

                // If the imageData is nil (i.e. if trying to save a UIImage directly or the image was transformed on download)
                // and the image has an alpha channel, we will consider it PNG to avoid losing the transparency
                int alphaInfo = CGImageGetAlphaInfo(image.CGImage);
                BOOL hasAlpha = !(alphaInfo == kCGImageAlphaNone ||
                                  alphaInfo == kCGImageAlphaNoneSkipFirst ||
                                  alphaInfo == kCGImageAlphaNoneSkipLast);
                BOOL imageIsPng = hasAlpha;

                // But if we have an image data, we will look at the preffix
                if ([imageData length] >= [kPNGSignatureData length]) {
                    imageIsPng = ImageDataHasPNGPreffix(imageData);
                }

                if (imageIsPng) {
                    data = UIImagePNGRepresentation(image);
                }
                else {
                    data = UIImageJPEGRepresentation(image, (CGFloat)1.0);
                }
#else
                data = [NSBitmapImageRep representationOfImageRepsInArray:image.representations usingType: NSJPEGFileType properties:nil];
#endif
            }

            [self storeImageDataToDisk:data forKey:key];
        });
    }
```
此处的代码由于recalculate这个值为NO，故会直接跳过中间的if语句，把data拿去执行写磁盘的操作，否则的话写磁盘的图片文件就也是解码过后的了，那么磁盘文件的大小也就不再可靠了。


后记：

关于SD的缓存方式，AFNetworking的作者曾经吐槽过。

目测使用系统自带的缓存方案就很不错的样子。。。