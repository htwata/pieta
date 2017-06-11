---
layout: post
title:  "iOSでAdmobのBannerとInterstitial用のユーティリティ"
date:   2017-05-10 20:01:14 +0900
tags: ios swift xcode delegate admob 
permalink: /ios/swift/xcode/delegate/admob

---
```
Swift3.0.2 Xcode 8.2
```

```
BannerはSnapkitライブラリでレイアウトを設定しているため、
事前にインストールする必要がある
```
#### バナー

{% highlight Swift %}
//  GADBannerViewのインスタンス生成をカプセル化し、ViewContollerなどでコードの記述する量を減らすためのクラス

import Foundation
import UIKit
import SnapKit
import GoogleMobileAds


class GADCustomBannerView {
    
    
    static func newInstance(view: UIView) -> GADBannerView {
        // AdMob広告設定
        //var bannerView: GADBannerView = GADBannerView()
        let bannerView = GADBannerView(adSize:kGADAdSizeBanner)
        // AdMobで発行された広告ユニットIDを設定(Easy Playlist for Youtube)
      
        bannerView.adUnitID = "ca-app-pub-3073356833616528/2309536298"
 
        return bannerView
    }
    
    static func startGADRequest(_ bannerView: GADBannerView) -> GADBannerView {
        let gadRequest:GADRequest = GADRequest()
        
        // テスト用の広告を表示する時のみ使用（申請時に削除）

        bannerView.load(gadRequest)
        
        return bannerView
    }
    
    
    
}

//使用例(個別のViewContollerに拡張する必要がある)
extension ViewController: GADBannerViewDelegate {
    
    func setupGADBannerView(){
        
        let bannerView = GADCustomBannerView.newInstance(view: self.view)
 
        bannerView.delegate = self
        bannerView.rootViewController = self

        self.view.addSubview(GADCustomBannerView.startGADRequest(bannerView))
        
        
        /*
         SnapKitで制約を設定
         http://qiita.com/kukimo/items/efaa0bad04e15159a5a2
         https://developers.google.com/mobile-ads-sdk/docs/admob/ios/ad-events?hl=ja
         GADBannerView に制約を設定して、画面下部の中央に 320x50 のサイズで表示されるようにします。
         */
        bannerView.snp.makeConstraints { (make) -> Void in
            
           //https://github.com/SnapKit/SnapKit/issues/85
            //ツールバーの上に設置
            make.bottom.equalTo(toolBarTest.snp.top)
            //横幅を端末(親ビュー:view)と同じサイズ
            make.width.equalTo(view.snp.width)
            make.height.equalTo(50)
        }
    }
}
{% endhighlight % }

#### インタースティシャル

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