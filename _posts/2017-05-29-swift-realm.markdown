---
layout: post
title:  "Realmの基本的な使用方法"
date:   2017-06-10 20:01:14 +0900
tags: ios swift xcode library realm 
permalink: /ios/swift/xcode/realm
---
```
Swift3.0.2 Xcode 8.2
```

Carthage
```
github "realm/realm-cocoa"
```
RealmSwift.frameworkとRealm.frameworkの2つを導入する。


#### モデルクラスの作成

保存したいクラスを定義するときは、Object(RealmSwift)を拡張したクラスとして定義し、
フィールドの修飾子は、dynamic varとして指定する

{% highlight Swift %}
import RealmSwift

class RMVideoItem: Object {
    dynamic var videoId: String = ""
    dynamic var thumbnailUrl: String = ""
    dynamic var title: String = ""
    dynamic var videoText: String = ""
    dynamic var channelTitle: String = ""
    dynamic var channelId: String = ""
}
{% endhighlight %}

#### 保存・読み込み
初期化したrealmオブジェクトのwriteトランザクションの内部で保存・更新を行う。

```
let realm = try! Realm()
try! realm.write {
///保存したいオブジェクトの代入・メンバー変数の変更
}
```
{% highlight Swift %}
static func saveVideoItem(item: VideoItem){
        var hVideoItem = RMVideoItem()
        hVideoItem.title = item.title
        hVideoItem.videoId = item.videoId
        hVideoItem.thumbnailUrl = item.thumbnail
       
        let realm = try! Realm()
        try! realm.write {
            //addで保存したオブジェクト(RMVideoItem)は、
            //realm.objects(RMVideoItem)でResult<RMVideoItem>として取得される。
            realm.add(hVideoItem)
        } 
    }
{% endhighlight %}

アイテムの読み込みは、読み込む対象のクラス*(T : RealmSwift.Object)*を引数指定する
objects(T : RealmSwift.Object)メソッドで取得する。
このobjectsメソッドの返り値は、RealmSwift.Results<T>コレクションクラスになっており、Arrayと同じような使用方法ができる。
{% highlight Swift %}
@IBAction func loadButtonTapped(_ sender: AnyObject) {  
        let realm = try! Realm()
        //RMVideoItemを格納したResuts<RMVideoItem>を取得する。
        let results = realm.objects(RMVideoItem)
        print(results[1].videoId)
         
        }
{% endhighlight %}

#### 1対多数の関係

{% highlight Swift %}
//1(入れ子、再生リスト)
class RMPlaylist: Object {
     let itemList = List<RMVideoItem>()
}
//多数(VideoItem)
class RMVideoItem: Object {
    dynamic var videoId: String = ""
    dynamic var thumbnailUrl: String = “”
}
{% endhighlight %}

保存するには、
```
①VideoItemの入れ子であるRMPlaylistを保存し、
②その保存したRMPlaylistにVideoItemを入れる
```

{% highlight Swift %}
 static func saveVideoItem(item: VideoItem){
        var hVideoItem = RMVideoItem()
        hVideoItem.title = item.title
        hVideoItem.videoId = item.videoId
        hVideoItem.thumbnailUrl = item.thumbnail
       
        //保存する再生リストを読み込む
        let realm = try! Realm()
        let playlistResults = realm.objects(RMPlaylist)
       
        try! realm.write {
            //① 再生リストの要素を追加する.
            let playlistItem = RMPlaylist()
            realm.add(playlistItem)
           
            //② 再生リストの0番目の要素にアイテムを追加する。itemList: List<RMVideoItem>
            playlistResults[0].itemList.append(hVideoItem)
        }
       
    }

            //更新するときは、writeトランザクションの内部で行う。
        try! realm.write {
            //要素を更新 
            results[0].itemList[0].title = "Updated title name"
        }

            //削除
        try! realm.write {
            //削除
            results[0].itemList.remove(objectAtIndex: 0)
        }
{% endhighlight %}