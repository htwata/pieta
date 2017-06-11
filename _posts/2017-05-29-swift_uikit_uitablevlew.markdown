---
layout: post
title:  "UITableViewの備忘録(Custom Cell,セルの編集・削除・入れ替え・クリック時の処理, filtering)"
date:   2017-05-10 20:01:14 +0900
tags: ios swift xcode uikit uitableview
permalink: /ios/swift/xcode/uikit/uitableview

---
```
Swift3.0.2 Xcode 8.2
```

ここではStoryBoardとカスタムCellを用いて、
UITableViewを作成する。

カスタムCellとUITableViewクラスの関連付け
UITableViewプロトコルで用意されている関数などの
基本的な内容をまとめる

# UITableViewの基本的な作成方法(StoryBoard, Custom Cell使用)

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
```
・File's Owner → indntify inspecter
Custom Classで作成した*CustomViewCell*を指定

・CustomCell → Attributes inspecter
Style → *Custom*
identifier → *Cell* (tableView.dequeueReusableCell(withIdentifier: "Cell")で関連付け)
```

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

# セルの編集・削除・入れ替え・クリック時の処理などの操作

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
        //セルの編集を可能にする
        self.tableView.setEditing(true, animated: true)
        //複数選択を可能にする
        self.tableView.allowsMultipleSelectionDuringEditing = true
    }
    
    //指定されたセルの内容をUITableViewCellのインスタンスとして返す
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    
        if indexPath.section == 0 {
            
            //① tableView.registerと同じ引数をセットする。
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell") as! CustomViewCell
            
            //cell内のアイテムを変更する
            let titleLabel: UILabel = cell.viewWithTag(999) as! UILabel
            titleLabel.text = "DOOOOM"
            
            return cell
        }
    
        return  UITableViewCell()
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        //return items.count
        return 100
    }
    

    
    //セルをクリックした時の処理
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        //print("selected cell \(indexPath.row)")
        
        
        //プレイボタン
        if self.tableView.indexPathsForSelectedRows?.count == 1 {
            //self.playButton.isEnabled = true
            
        }else{
            //self.playButton.isEnabled = false
            
        }
        
        //ゴミ箱
        if self.tableView.indexPathForSelectedRow?.count == 0 {
         //   self.deleteItemButton.isEnabled = false
        }else{
       //     self.deleteItemButton.isEnabled = true
        }
        
        //        self.delegate?.changeMainTableViewResults(self.searchFilter!.getResults() as! NSMutableArray)
        //        self.delegate?.selectedFromSearchFilter(filterResults[indexPath.row])
        
    }
    
    //セルのクリックを解除した時の処理
    func tableView(_ tableView: UITableView, didDeselectRowAt indexPath: IndexPath){
        //プレイボタン
        if self.tableView.indexPathsForSelectedRows?.count == 1 {
          //  self.playButton.isEnabled = true
            
        }else{
            //self.playButton.isEnabled = false
            
        }
    }
    


    /*
     セルの移動設定
     */
    
    
    //移動できるようにする
    //このメソッドをオーバライドしないと、並び替えできない(右端に並び替えアイコンが出ない)
    func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool{
        return true
    }
    //並び替え実行後の処理
    //このメソッドをオーバライドしないと、並び替えできない
    func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath){
        
        // #201703191011 並び替え修正(更新がうまくいかない箇所を修正)
       // PlaylistHelper.sharedInstance.moveCell(moveRowAt: sourceIndexPath, to: destinationIndexPath)
        
        //TableViewを1度リセットして再読み込み
        //self.searchFilter?.reloadData()
        //self.filterResults.removeAll()
        //self.filterResults = self.searchFilter!.getResults()
        //self.self.tableView.reloadData()
        
    }
    
    
    /*
     左端のボタン設定(self.tableView.allowsMultipleSelectionDuringEditing = true)
     */
    
    //falseにすると編集(削除ボタン)が表示されなくなる
    //注意：この関数をオーバーライドすると、複数選択(青色のチェックボックス)ができなくなる
    func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool{
        
        return false
    }

    //左端にボタン(マイナスボタンやプラスボタン)を表示している時に、インデントを挿入するかどうか
    func tableView(_ tableView: UITableView, shouldIndentWhileEditingRowAt indexPath: IndexPath) -> Bool{
        return false
    }

    //以下のの関数で、左端に赤いマイナスボタンもしくは緑色のプラスボタン表示
    //http://stackoverflow.com/questions/3020922/is-there-any-way-to-hide-delete-button-while-editing-uitableview
    func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCellEditingStyle{
        //return .insert //プラスボタン
        return .delete //デリートボタン
        //return .none//何も表示しない
    }

    // 編集ボタンのカスタマイズButtonを拡張する.
    func tableView(_ tableView: UITableView, editActionsForRowAt: IndexPath) -> [UITableViewRowAction]? {
        
        //並び替えボタン21C5
        let arrowButton: UITableViewRowAction = UITableViewRowAction(style: .normal, title: "\u{21C5}") { (action, index) -> Void in
            
            tableView.isEditing = true
            print("share")
            
        }
        arrowButton.backgroundColor = UIColor.brown
        
        // heartボタン.
        let heartButton: UITableViewRowAction = UITableViewRowAction(style: .normal, title: "\u{2665}") { (action, index) -> Void in
            
            tableView.isEditing = false
            print("share")
            
        }
        heartButton.backgroundColor = UIColor.blue
        
        //U+25B6
        // Archiveボタン.
        let playButton: UITableViewRowAction = UITableViewRowAction(style: .normal, title: "\u{25B6}") { (action, index) -> Void in
            
            tableView.isEditing = false
            print("archive")
            
        }
        playButton.backgroundColor = UIColor.green
        
        // Deleteボタン.
        let myDeleteButton: UITableViewRowAction = UITableViewRowAction(style: .normal, title: "\u{2A2F}") { (action, index) -> Void in
            
            tableView.isEditing = false
            print("delete")
            
        }
        myDeleteButton.backgroundColor = UIColor.red
        //注意:左から順に登録されるらしい
        return [myDeleteButton, heartButton, playButton]
        
    }

}
{% endhighlight %}
# UISearchBarと組み合わせて、cellをフィルタリングする

## フィルタリング処理用クラスの作成

・ArrayFilter.swift(extendとして作成するのも可)
{% highlight Swift %}
import Foundation

class ArrayFilter {

    static func filterPartialMatchQuery(array: Array<AnyObject>, word: String) -> Array<AnyObject>{
        //1. searchQueryの検索条件を保持するオブジェクト
        let searchQuery = "name CONTAINS[c] %@"
    
        let pred:NSPredicate = NSPredicate(format: searchQuery, word)
    
        //searchQuery条件にマッチする配列をフィルタリング
        let result: Array<AnyObject>? = (array as NSArray).filtered(using: pred) as? Array<AnyObject>
    
        return result!
    }
 
    static func filterOrderByAscending(array: Array<AnyObject>, sortKey: String) -> Array<AnyObject>{
       
            // http://chris.eidhof.nl/post/sort-descriptors-in-swift/
            //並び替えの基準となる変数をkey, 上昇なら ascending: true
            // let ageDiscriptor = NSSortDescriptor(key: "age", ascending: false)
            let nameDiscriptor = NSSortDescriptor(key: sortKey, ascending: true)
        
            let result = (array as NSArray).sortedArray(using: [nameDiscriptor])
            
            return result as Array<AnyObject>
    }
    
    static func filterOrderByDescending(array: Array<AnyObject>, sortKey: String) -> Array<AnyObject>{
        
        // http://chris.eidhof.nl/post/sort-descriptors-in-swift/
        //並び替えの基準となる変数をkey, 上昇なら ascending: true
        // let ageDiscriptor = NSSortDescriptor(key: "age", ascending: false)
        let nameDiscriptor = NSSortDescriptor(key: sortKey, ascending: false)
        
        let result = (array as NSArray).sortedArray(using: [nameDiscriptor])
        
        return result as Array<AnyObject>
    }
    
    //複数キーワード(スペース区切り)
    static func filterMultipleWords(array: Array<AnyObject>, words: String) -> Array<AnyObject>{
        
        //曖昧条件
        let query = "name CONTAINS[c] %@"
        
        //空白ごとに配列の要素を追加する
        var stringArray = words.components(separatedBy: NSCharacterSet.whitespaces)
        print("stringArray = %@",stringArray.count)
        
        var predicate:Array<NSPredicate> = []
        
        for i in 0..<stringArray.count{
            print(stringArray[i])
            let item = NSPredicate(format: query, stringArray[i])
            predicate.append(item)
        }

        //OR条件
        let compoundedPredicate = NSCompoundPredicate(orPredicateWithSubpredicates: predicate)
        
        //searchQuery条件にマッチする配列をフィルタリング
        let result: Array<AnyObject>? = (array as NSArray).filtered(using: compoundedPredicate) as? Array<AnyObject>

        return result!

    }
}
{% endhighlight %}

・ViewController.swift
{% highlight Swift %}
import UIKit
import Foundation

class ViewController: UIViewController,UITableViewDelegate, UITableViewDataSource,UISearchBarDelegate {
    
    //サンプル:@IBOutlet weak var tableView: UITableView!
    @IBOutlet weak var tableView: UITableView!
    @IBOutlet weak var searchBar: UISearchBar!

    //サンプルアイテムのコレクション
    
    //フィルタリング処理後の配列。これをTableViewと連携させ、実際に表示させる。
    var searchResult: Array<AnyObject>  = []
    //元の配列(これ自体は変更しない)
    var nekoSampleArray: Array<AnyObject> = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        //self.nekoSampleArray = [neko0, neko1, neko2, neko3, neko4, neko5, neko6, neko7, neko8, neko9, neko10, neko11, neko12, neko13, neko14, neko15, neko16]
        for i in 0..<commandoSample.count{
            self.nekoSampleArray.append(Neko(name: commandoSample[i], age: i))
        }
        
        searchResult = nekoSampleArray
        
        initSearchController()
        initTableView()
    }
    
    //デリゲートの登録とXibの読み込み
    func initTableView(){
        tableView.delegate = self
        tableView.dataSource = self
        var nib = UINib(nibName: "CustomViewCell", bundle: nil)
        
        //① tableView.dequeueReusableCell(withIdentifier: )と引数が対応関係にある。
        self.tableView.register(nib, forCellReuseIdentifier: "Cell")
    }

    
    func initSearchController(){
        
        //結果表示用のビューコントローラーに自分を設定する。
        searchBar.delegate = self
        //空白でも検査ボタンが押せる
        searchBar.enablesReturnKeyAutomatically = false
        searchBar.setShowsCancelButton(true, animated: true)
        
    }
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
    
    //指定されたセルの内容をUITableViewCellのインスタンスとして返す
    //cellForRowAtは外部引数(詳解Swift p40)
    //アンダースコアは内部引数の省略(詳解Swift p41-42)
    //TableViewについては、iOSアプリ開発p335を参照
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        if indexPath.section == 0 {
            
            //① tableView.registerと同じ引数をセットする。
            var cell = tableView.dequeueReusableCell(withIdentifier: "Cell") as! CustomViewCell
            
            //cell内のアイテムを変更する
            var titleLabel: UILabel = cell.viewWithTag(100) as! UILabel
            
            
            // Array<Item>の場合
            //titleLabel.text = sampleArray[indexPath.row].title
            print("tableView")
            titleLabel.text = (self.searchResult[indexPath.row] as! Neko).name
            
            return cell
        }
        
        return  UITableViewCell()
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        //return items.count
        //return commandoSample.count-1
        return searchResult.count
    }
    
    /*
     テキストが入力されるたびに呼ばれる(確定する前)
    */
    func searchBar(_ searchBar: UISearchBar, shouldChangeTextIn range: NSRange, replacementText text: String) -> Bool {
        
        print(searchBar.text!)
        // 確定前の文字入力を検知します
        // return trueして文字入力が確定した後にsearchBarの文字を取得する処理を遅延実行します
        print("Before")
        
        /*遅滞処理の中でフィルタリング・更新処理を行わないと、
        タイミングが早すぎて確定前のsearch.textが取得できない*/
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.1){
            print("Dispatche?")
            if(searchBar.text == ""){
                
                print("not text")
                self.searchResult = self.nekoSampleArray
                
            }else if(searchBar.text?.contains(" "))!{
                
                print(" Multi words")
                self.searchResult = ArrayFilter.filterMultipleWords(array: self.nekoSampleArray, words: searchBar.text!)

            }else{
                
                self.searchResult = ArrayFilter.filterPartialMatchQuery(array: self.nekoSampleArray, word: searchBar.text!)
            }
            self.tableView.reloadData()
        }//DispatchQueu.main.asyncAfter
        return true
    }
    
    /*
     Searchボタンが押された時に呼ばれる
     */
    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        self.view.endEditing(true)
        print("SearchButton")
    }
    
    /*
     テキストが確定毎に呼ばれる
     */
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        print("Daemon")
        //self.tableView.reloadData()
    }
    
    /*
     Cancelボタンが押された時に呼ばれる
     */
    func searchBarCancelButtonClicked(_ searchBar: UISearchBar) {
        print()
        self.searchResult = ArrayFilter.filterOrderByDescending(array: nekoSampleArray, sortKey: "name")
        self.tableView.reloadData()

    }
}
{% endhighlight %}