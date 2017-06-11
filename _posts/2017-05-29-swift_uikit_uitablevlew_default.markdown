---
layout: post
title:  "UITableViewの基本的な作成方法(StoryBoard, Custom Cell使用)"
date:   2017-05-10 20:01:14 +0900
tags: ios swift xcode uikit uitableview
permalink: /ios/swift/xcode/uikit/uitableview/default

---
```
Swift3.0.2 Xcode 8.2
```

ここではStoryBoardとカスタムCellを用いて、
UITableViewを作成する。

カスタムCellとUITableViewクラスの関連付け
UITableViewプロトコルで用意されている関数などの
基本的な内容をまとめる

## CustomViewCell.swiftの準備

新規ファイル作成から*Cocoa Touch Class*で
"Subclass of "項目から*UITableViewCell*を選択
(also create Xibにもチェックを入れる)
・CustomViewCell.swift
{% highlight Swift %}
import UIKit

class CustomViewCell: UITableViewCell {
    
      // Outlet接続の必要はない

    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)

        // Configure the view for the selected state
    }
   
    func setCell(videoItem: VideoItem){
       
    }
   
}
{% endhighlight %}

#### CustomViewCell.xibの設定

swiftファイルとxibファイルを関連付けするためにストーリーボードで以下の設定を行う

・File's Owner → indntify inspecter
Custom Classで作成した*CustomViewCell*を指定

・CustomCell → Attributes inspecter
Style → *Custom*
identifier → *Cell* (tableView.dequeueReusableCell(withIdentifier: "Cell")で関連付け)

## UITableView表示用クラスの作成とカスタムセルの関連付け

新規ファイル作成からUITableViewを作成
UITableViewとカスタムセルの関連付けは以下の関数で行う
{% highlight Swift %}
//カスタムセルが定義されているswiftファイル名(nibName)
var nib = UINib(nibName: "CustomViewCell", bundle: nil)
//Attributes inspecterで指定したidentifier名
self.tableView.register(nib, forCellReuseIdentifier: "Cell")
{% endhighlight %}

・ViewController.swift
{% highlight Swift %}
import UIKit

class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {

    
    //ストーリーボードからアウトレット接続
    @IBOutlet weak var tableView: UITableView!
   
    //
    var items: Array<String> = ["Doom", "Han"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        initTableView()
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
    
    //初期化デリゲートの登録とXibの読み込み
    func initTableView(){
        tableView.delegate = self
        tableView.dataSource = self
        
        //作成したカスタムセルのファイル名を指定する
        var nib = UINib(nibName: "CustomViewCell", bundle: nil)

        self.tableView.register(nib, forCellReuseIdentifier: "Cell")
    }
    
    //指定されたセルの内容をUITableViewCellのインスタンスとして返す
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    
        if indexPath.section == 0 {
            
            //① tableView.registerと同じ引数をセットする。
            var cell = tableView.dequeueReusableCell(withIdentifier: "Cell") as! CustomViewCell
            
            //cell内のアイテムを変更する
            var titleLabel: UILabel = cell.viewWithTag(999) as! UILabel
            titleLabel.text = "DOOOOM"
            
            return cell
        }
    
        return  UITableViewCell()
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        //return items.count
        return 100
    }


}
{% endhighlight %}


{% highlight Swift %}
{% endhighlight %}