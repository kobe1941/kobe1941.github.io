---
layout: post
title: "UIWebView网络图片宽度自适应"
date: 2015-03-29 21:42:36 +0800
comments: true
categories: iOS开发
keywords: UIWebView 图片 宽度自适应
---
###本文介绍UIWebView在加载图片时，如何使图片宽度自适应屏幕。

<!--more-->

该方案适用于纯图片介绍商品详情，让图片自适应屏幕宽度。

###1.首先设置网页宽度自适应屏幕，并避免其滚动：

``` Objective-c

   _webView.scalesPageToFit = YES; // 宽度自适应
   _webView.scrollView.scrollEnabled = NO; // 禁止滚动
   
```

###2.其次，拿到html字符串之后，拼接一串处理的html代码即可：
```
NSString *const RESIZE_PICTURE_STRING = @"<script>"
"var imgs = document.getElementsByTagName('img');"
"for(var i = 0; i < imgs.length; i++){"
"imgs[i].setAttribute(\"width\",\"100%\");"
"imgs[i].removeAttribute(\"height\");"
"imgs[i].removeAttribute(\"style\");"
"}"
"</script>";
```

```
[_htmlString appendString:RESIZE_PICTURE_STRING];
```

####注：网上也有另一种方式来处理，但是纯图片的显示效果不佳。

```
- (void)webViewDidFinishLoad:(UIWebView *)webView
{
	NSLog(@"网页加载完成");
	
	
	[webView stringByEvaluatingJavaScriptFromString:
	@"var script = document.createElement('script');"
	"script.type = 'text/javascript';"
	"script.text = \"function ResizeImages() { "
	"var myimg,oldwidth;"
	"var maxwidth = 980;" //缩放系数
	"for(i = 0;i < document.images.length;i++){"
	"myimg = document.images[i];"
	"if(myimg.width != maxwidth){" // 注意改这个符号，小于或者大于
	"oldwidth = myimg.width;"
	"myimg.width = maxwidth;"
	// "myimg.height *= (maxwidth/oldwidth);"
	"}"
	"}"
	"}\";"
	"document.getElementsByTagName('head')[0].appendChild(script);"];
	
	[webView stringByEvaluatingJavaScriptFromString:@"ResizeImages();"];
	// 高度微调
	[[webView stringByEvaluatingJavaScriptFromString:@"document.documentElement.scrollHeight"] intValue];
	
	[webView sizeToFit];

}
```