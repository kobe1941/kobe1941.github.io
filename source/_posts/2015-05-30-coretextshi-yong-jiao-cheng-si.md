---
layout: post
title: "CoreText使用教程四"
date: 2015-05-30 21:37:21 +0800
comments: true
categories: iOS开发
keywords: CoreText 省略号  
description: 
---

本篇文章为CoreText系列的第四篇，接着上一篇，讨论在纯文本排版时，当控件高度不够需要显示省略号的问题。

本文介绍了将省略号放在可以显示的最后一个字符或者最后一行的任何位置的实现过程。

实现代码在github的[仓库](https://github.com/kobe1941/HFCoreTextDemo)

实现效果见下图：

省略号放到最后一个字符

![](/images/2015/05/30/CoreText_1.png)



省略号放置在最后一行约1/3行处

![](/images/2015/05/30/CoreText_2.png)


<!--more-->

上篇博文已经实现了一行一行的绘制文本，但有些产品的需求是暂时显示一些简要的信息，用户点击后再进入到详情页显示完全。这时候则需要在文本的显示末端添加一个省略号。

这种需求在微博和朋友圈都有，一条微博或者朋友圈信息较长时，为方便其他用户阅读，往往需要截断，只显示部分信息。

本次实现基本参照了[Nimbus](https://github.com/jverkoey/nimbus/blob/master/src/attributedlabel/src/NIAttributedLabel.m)的实现，我按照自己的理解在代码中加入了一些注释，如果有不对的地方还请指出。

大部分代码跟之前类似，以下是绘制部分的代码，代码中注释有说明怎么实现改变省略号的位置：

```Objective-C

#pragma mark - 一行一行绘制，行高确定，高度不够时加上省略号
- (void)drawRectWithLineByLineAlignmentAndEllipses
{
    
    // 1.创建需要绘制的文字
    NSMutableAttributedString *attributed = [[NSMutableAttributedString alloc] initWithString:self.text];
    
    // 2.设置行距等样式
    [[self class] addGlobalAttributeWithContent:attributed font:self.font];
    
    
    self.textHeight = [[self class] textHeightWithText:self.text width:CGRectGetWidth(self.bounds) font:self.font type:self.drawType];
    
    // 3.创建绘制区域，path的高度对绘制有直接影响，如果高度不够，则计算出来的CTLine的数量会少一行或者少多行
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathAddRect(path, NULL, CGRectMake(0, 0, CGRectGetWidth(self.bounds), self.textHeight*2));
    
    // 4.根据NSAttributedString生成CTFramesetterRef
    CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attributed);
    
    CTFrameRef ctFrame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attributed.length), path, NULL);
    
    // 重置高度
    CGFloat realHeight = self.textHeight;
    // 绘制全部文本需要的高度大于实际高度则调整，并加上省略号
    if (realHeight > CGRectGetHeight(self.frame))
    {
        realHeight = CGRectGetHeight(self.frame);
    }

    NSLog(@"realHeight = %f",realHeight);
    
    // 获取上下文
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    
    // 转换坐标系
    CGContextSetTextMatrix(contextRef, CGAffineTransformIdentity);
    CGContextTranslateCTM(contextRef, 0, realHeight); // 这里跟着调整
    CGContextScaleCTM(contextRef, 1.0, -1.0);
    
    // 这里可调整可不调整
    CGPathAddRect(path, NULL, CGRectMake(0, 0, CGRectGetWidth(self.bounds), realHeight));
    
    // 一行一行绘制
    CFArrayRef lines = CTFrameGetLines(ctFrame);
    CFIndex lineCount = CFArrayGetCount(lines);
    CGPoint lineOrigins[lineCount];
    
    // 把ctFrame里每一行的初始坐标写到数组里，注意CoreText的坐标是左下角为原点
    CTFrameGetLineOrigins(ctFrame, CFRangeMake(0, 0), lineOrigins);

    
    CGFloat frameY = 0;
    
    
    for (CFIndex i = 0; i < lineCount; i++)
    {
        // 遍历每一行CTLine
        CTLineRef line = CFArrayGetValueAtIndex(lines, i);
        
        
        CGFloat lineAscent;
        CGFloat lineDescent;
        CGFloat lineLeading; // 行距
        // 该函数除了会设置好ascent,descent,leading之外，还会返回这行的宽度
        CTLineGetTypographicBounds(line, &lineAscent, &lineDescent, &lineLeading);

        CGPoint lineOrigin = lineOrigins[i];
        
        // 微调Y值，需要注意的是CoreText的origin的Y值是在baseLine处，而不是下方的descent。
        CGFloat lineHeight = self.font.pointSize * kPerLineRatio;
        
        // 调节self.font.descender该值可改变文字排版的上下间距，此处下间距为0
        frameY = realHeight - (i + 1)*lineHeight - self.font.descender;
        
        NSLog(@"frameY = %f",frameY);
        
        lineOrigin.y = frameY;
        
        // 调整坐标
        CGContextSetTextPosition(contextRef, lineOrigin.x, lineOrigin.y);
        
        // 反转坐标系
        frameY = realHeight - frameY;
        
        NSLog(@"realHeight = %f,font.descender = %f",realHeight,self.font.descender);
        NSLog(@"反转后的坐标 y = %f",frameY);
        
        // 行高
        CGFloat heightPerLine = self.font.pointSize * kPerLineRatio;
        
        if (realHeight - frameY > heightPerLine)
        {
            CTLineDraw(line, contextRef);
            
            NSLog(@"一行一行的画 i = %ld",i);
            
        } else
        {
            NSLog(@"最后一行");
            
            // 最后一行，加上省略号
            static NSString* const kEllipsesCharacter = @"\u2026";
            
            CFRange lastLineRange = CTLineGetStringRange(line);
            
            // 一个emoji表情占用两个长度单位
            NSLog(@"range.location = %ld,range.length = %ld,总长度 = %ld",lastLineRange.location,lastLineRange.length,attributed.length);
            
            if (lastLineRange.location + lastLineRange.length < (CFIndex)attributed.length)
            {
                // 这一行放不下所有的字符（下一行还有字符），则把此行后面的回车、空格符去掉后，再把最后一个字符替换成省略号
                
                CTLineTruncationType truncationType = kCTLineTruncationEnd;
                NSUInteger truncationAttributePosition = lastLineRange.location + lastLineRange.length - 1;
                
                // 拿到最后一个字符的属性字典
                NSDictionary *tokenAttributes = [attributed attributesAtIndex:truncationAttributePosition
                                                                     effectiveRange:NULL];
                // 给省略号字符设置字体大小、颜色等属性
                NSAttributedString *tokenString = [[NSAttributedString alloc] initWithString:kEllipsesCharacter
                                                                                  attributes:tokenAttributes];
                
                // 用省略号单独创建一个CTLine，下面在截断重新生成CTLine的时候会用到
                CTLineRef truncationToken = CTLineCreateWithAttributedString((__bridge CFAttributedStringRef)tokenString);
                
                // 把这一行的属性字符串复制一份，如果要把省略号放到中间或其他位置，只需指定复制的长度即可
                NSUInteger copyLength = lastLineRange.length/3;
                
                NSMutableAttributedString *truncationString = [[attributed attributedSubstringFromRange:NSMakeRange(lastLineRange.location, copyLength)] mutableCopy];
                
                if (lastLineRange.length > 0)
                {
                    // Remove any whitespace at the end of the line.
                    unichar lastCharacter = [[truncationString string] characterAtIndex:copyLength - 1];
                    
                    // 如果复制字符串的最后一个字符是换行、空格符，则删掉
                    if ([[NSCharacterSet whitespaceAndNewlineCharacterSet] characterIsMember:lastCharacter])
                    {
                        [truncationString deleteCharactersInRange:NSMakeRange(copyLength - 1, 1)];
                    }
                }
                
                // 拼接省略号到复制字符串的最后
                [truncationString appendAttributedString:tokenString];
                
                // 把新的字符串创建成CTLine
                CTLineRef truncationLine = CTLineCreateWithAttributedString((__bridge CFAttributedStringRef)truncationString);
                
                // 创建一个截断的CTLine，该方法不能少，具体作用还有待研究
                CTLineRef truncatedLine = CTLineCreateTruncatedLine(truncationLine, self.frame.size.width, truncationType, truncationToken);
                
                if (!truncatedLine)
                {
                    // If the line is not as wide as the truncationToken, truncatedLine is NULL
                    truncatedLine = CFRetain(truncationToken);
                }
                
                CFRelease(truncationLine);
                CFRelease(truncationToken);
                
                CTLineDraw(truncatedLine, contextRef);
                CFRelease(truncatedLine);
                
            } else
            {
                
                // 这一行刚好是最后一行，且最后一行的字符可以完全绘制出来
                CTLineDraw(line, contextRef);
            }
            
            // 跳出循环，避免绘制剩下的多余的CTLine
            break;
            
        }
        
        
        

    }
    
    
    
    CFRelease(path);
    CFRelease(framesetter);
    CFRelease(ctFrame);

}


```

同时为了兼容这次的代码实现，控制器部分进行了重构：

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    NSString *content = @"我自横刀向天笑，去留肝胆两昆仑。--谭嗣同同学你好啊。This is my first CoreText demo,how are you ?I love three things,the sun,the moon,and you.the sun for the day,the moon for the night,and you forever.😳😊😳😊😳😊😳去年今日此门中，人面桃花相映红。人面不知何处去，桃花依旧笑春风。😳😊😳😊😳😊😳少年不知愁滋味，爱上层楼，爱上层楼，为赋新词强说愁。56321363464.而今识尽愁滋味，欲说还休，欲说还休，却道天凉好个秋。123456，7890，56321267895434。缺月挂疏桐，漏断人初静。谁见幽人独往来，缥缈孤鸿影。惊起却回头，有恨无人省。捡尽寒枝不肯栖，寂寞沙洲冷。";
   
    self.coreTextView.backgroundColor = [UIColor grayColor];
    self.coreTextView.font = [UIFont systemFontOfSize:15];
    self.coreTextView.text = content;
    
    self.coreTextView.drawType = HFDrawTextWithEllipses; // 设置该值即可切换
    
    // 此时self.coreTextView的宽度为320，为了在iPhone6上计算准确，使用屏幕宽度
    CGFloat realWidth = [UIScreen mainScreen].bounds.size.width;
    
    CGFloat height = [HFCoreTextView textHeightWithText:content width:realWidth font:self.coreTextView.font type:self.coreTextView.drawType];

    // 在这里控制显示的行数
    CGFloat maxHeight = (self.coreTextView.font.pointSize*kPerLineRatio)*6;
    
    if (height > maxHeight && self.coreTextView.drawType == HFDrawTextWithEllipses)
    {
        height = maxHeight;
    }
    
    NSLog(@"height = %f",height);
    
    self.contentViewHeightConstraint.constant = height;

}


```

参考链接：

1.[猿题库iOS客户端的技术细节（三）：基于CoreText的排版引擎](http://blog.devtang.com/blog/2013/10/21/the-tech-detail-of-ape-client-3/)

2.[Nimbus的NIAttributedLabel实现](https://github.com/jverkoey/nimbus/blob/master/src/attributedlabel/src/NIAttributedLabel.m)