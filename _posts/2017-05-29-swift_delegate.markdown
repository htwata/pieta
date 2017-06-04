---
layout: post
title:  "[Swift3.0.2][Xcode][delegate]delegateの使い方サンプル"
date:   2017-05-10 20:01:14 +0900
tags: ios swift xcode delegate
permalink: /swift/xcode/delegate
---
```
Swift 3.0.2 Xcode 8.2
```

そもそもデリゲートとは？
デリゲートは他のクラスから関数を遠隔操作させる仕組みである。

デリゲートを呼び出すクラスA(「funcTestを実行して」と委託する) → Delegate(仲介) →　実際にfuncTestを実行するクラスB
呼び出す側のクラスAは、委託先を意識することなく処理を委託できるため、クラスB以外のクラスCでも処理できる。

デリゲート実装を考えるときは、上記の3つを考慮して作成する。

```
・委託するクラス         →  RelatedVideoItemExtractor (class)
・デリゲートプロトコル    →  RelatedVideoItemDelegate (protocol)
・委託されるクラス       →  ViewController (class)
```

● delegateを行うクラス(遠隔操作を実行する)
{% highlight Swift %}
@objc protocol SampleViewDelegate {
  func setColor(color: UIColor)
}

class SampleView: UIView {
   //このdelegateオブジェクトが仲介役となり、SampleViewDelegateプロトコルを実装しているclassに対して遠隔操作する
  weak var delegate: SampleViewDelegate?

  @IBAction func color1WasTapped() {
    //ViewControllerにあるsetColor関数を実行させる
    self.delegate?.setColor(color: UIColor(red: 0.89, green: 0.59, blue: 0.59, alpha: 1))
  }
{% endhighlight %}

● delegateで呼び出され、SampleViewDelegateのsetColorを実際に実行するクラス(主にViewControllerで使用することが多い？)
{% highlight Swift %}
  @IBAction func buttonShowWasTapped() {
    var items = [CustomizableActionSheetItem]()

    // First view
    if let sampleView = UINib(nibName: "SampleView", bundle: nil).instantiate(withOwner: self, options: nil)[0] as? SampleView {
      //呼び出し元のdelegateと紐付ける処理
      sampleView.delegate = self


extension ViewController: SampleViewDelegate {
   //SampleViewで実行させるクラス
  func setColor(color: UIColor) {
    if let actionSheet = self.actionSheet {
      actionSheet.dismiss()
    }
    self.view.backgroundColor = color
  }
}
{% endhighlight %}