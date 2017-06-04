---
layout: post
title:  "正規表現にマッチした回数や文字列を得る"
date:   2017-05-10 20:01:14 +0900
tags: ios swift xcode delegate number match string
permalink: /ios/swift/xcode/numberOfMatchInString

---
```
Swift3.0.2 Xcode 8.2
```

[参考サイト]: http://joyplot.com/documents/2016/08/16/swift正規表現にマッチした回数や文字列を得る/

・NumberOfMatchingString.swift
{% highlight Swift %}
import Foundation

class NumberOfMatchingString {
    
    //isCaseInsensitive = true -> 大文字小文字を無視/
    static func getMatchCount(targetString: String, pattern: String, isCaseInsensitive: Bool) -> Int {
        
        do {
            //大文字小文字の区別を無視するオプション .caseInsensitive
            if(isCaseInsensitive){
                let regex = try NSRegularExpression(pattern: pattern, options: [.caseInsensitive])
                let targetStringRange = NSRange(location: 0, length: (targetString as NSString).length)
                return regex.numberOfMatches(in: targetString, options: [], range: targetStringRange)
            }else{
                let regex = try NSRegularExpression(pattern: pattern)
                let targetStringRange = NSRange(location: 0, length: (targetString as NSString).length)
                return regex.numberOfMatches(in: targetString, options: [], range: targetStringRange)
            }
            
        } catch {
            print("error: getMatchCount")
        }
        return 0
    }
    
}
{% endhighlight %}