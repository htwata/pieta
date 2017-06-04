---
layout: post
title:  "DelegateをでAdmobのInterstitial広告を遠隔実行するExtension"
date:   2017-05-10 20:01:14 +0900
tags: ios swift xcode delegate admob interstitial
permalink: /ios/swift/xcode/delegate/admob/interstitial

---
```
Swift3.0.2 Xcode 8.2
```

・GADInterstitialView.swift 
{% highlight Swift %}
@objc protocol SampleViewDelegate {
import Foundation
import GoogleMobileAds

protocol GADInterstitialExcuteDelegate {
    func showInterstitial()
}

extension ViewController: GADInterstitialDelegate, GADInterstitialExcuteDelegate {
    
    //viewDidLoadなどで読み込んでおく
    func initGADInterstitial(_ adID: String) {
        
        interstitial = GADInterstitial(adUnitID: adID)
        interstitial!.delegate = self
        interstitial!.load(GADRequest())

    }
    
    //protocol: GADInterstitialExcuteDelegate => delegateを介してインタースティシャスを実行する
    func showInterstitial(){
        
        if self.interstitial == nil {
            return
        }
        
        if(self.interstitial!.isReady == true){
            //ViewControllerに対してintersitialを表示する
            interstitial!.present(fromRootViewController: self)
        }
        
    }
    
}
{% endhighlight %}

・ViewController.swift(interstitialオブジェクトを初期化)
{% highlight Swift %}
import GoogleMobileAds
class ViewController: UIViewController {
    
  var interstitial: GADInterstitial?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.initGADInterstitial("ca-app-pub-xxxxxxxxxxxxxxxx")
    }
{% endhighlight %}

・ViewControllerから別のUIViewを作成し、delegateの紐付け
{% highlight Swift %}
extension BMPlayerViewController {
    
    func showPlaylistRegisterModalView(){
        var items = [CustomizableActionSheetItem]()
        
        // ViewControllerから別のUIViewクラスを生成し、UIViewクラスからdelegateを実行するため、admobDelegate = selfを行う
        if let playlistModalView = UINib(nibName: "PlaylistRegistModalView", bundle: nil).instantiate(withOwner: self, options: nil)[0] as? PlaylistRegistModalView {
            
            //protocol: GADInterstitialExcuteDelegate
            playlistModalView.admobDelegate = self
            
            items.append(sampleViewItem)
        }
{% endhighlight %}

・UIViewから実行
{% highlight Swift %}
class TestUIView: UIView {
    //prptocol: GADIntersititialExcuteDelegate
    var admobDelegate: GADInterstitialExcuteDelegate?
    
    /* Navigation Controller Items */
    
    @IBAction func doSomething(_ sender: Any) {
        self.admobDelegate?.showInterstitial()
    }
    
    
}
{% endhighlight %}