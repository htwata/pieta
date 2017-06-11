---
layout: post
title:  "UICollectionViewの備忘録(Custom Cell, セル選択時にカラーマスク)"
date:   2017-05-10 20:01:14 +0900
tags: ios swift xcode uikit uicollectionview
permalink: /ios/swift/xcode/uikit/uicollectionview

---
```
Swift3.0.2 Xcode 8.2
```
ここではUICollectionViewクラスの基本的な使用法をまとめる

セットアップ方法(UITableViewとほとんど同じ手順)

```
1 XibによるCustomCellの作成と関連づけ(StoryBoard)
2 CollectionViewのレイアウト作成(StoryBoard)
3 Cellの初期化
4 ViewControllerをUICollectionViewDataSourceクラスのサブクラス化(専用extensionの作成)

1 XibによるCustomCellの作成と関連づけ(StoryBoard)
New -> File -> Cocoa Touch ClassからカスタムCell用のXibファイルを作成する
(Also create XIB Fileにもチェックを入れる)
```

# セルタップ時にセルにカラーマスクを適応させる

#### セルにカラーマスクをかけるためのクラス

・UIImageColorize.swift(UIColorクラスのエクステンションで、カラーフィルターを行うことができる)
{% highlight Swift %}
import UIKit

extension UIImage {
    
    // colorize image with given tint color
    // this is similar to Photoshop's "Color" layer blend mode
    // this is perfect for non-greyscale source images, and images that have both highlights and shadows that should be preserved
    // white will stay white and black will stay black as the lightness of the image is preserved
    func tint(tintColor: UIColor) -> UIImage {
        
        return modifiedImage { context, rect in
            // draw black background - workaround to preserve color of partially transparent pixels
            context.setBlendMode(.normal)
            UIColor.black.setFill()
            context.fill(rect)
            
            // draw original image
            context.setBlendMode(.normal)
            context.draw(self.cgImage!, in: rect)
            
            // tint image (loosing alpha) - the luminosity of the original image is preserved
            context.setBlendMode(.color)
            tintColor.setFill()
            context.fill(rect)
            
            // mask by alpha values of original image
            context.setBlendMode(.destinationIn)
            context.draw(self.cgImage!, in: rect)
        }
    }
    
    // fills the alpha channel of the source image with the given color
    // any color information except to the alpha channel will be ignored
    func fillAlpha(fillColor: UIColor) -> UIImage {
        
        return modifiedImage { context, rect in
            // draw tint color
            context.setBlendMode(.normal)
            fillColor.setFill()
            context.fill(rect)
//            context.fillCGContextFillRect(context, rect)
            
            // mask by alpha values of original image
            context.setBlendMode(.destinationIn)
            context.draw(self.cgImage!, in: rect)
        }
    }
    
    private func modifiedImage( draw: (CGContext, CGRect) -> ()) -> UIImage {
        
        // using scale correctly preserves retina images
        UIGraphicsBeginImageContextWithOptions(size, false, scale)
        let context: CGContext! = UIGraphicsGetCurrentContext()
        assert(context != nil)
        
        // correctly rotate image
        context.translateBy(x: 0, y: size.height)
        context.scaleBy(x: 1.0, y: -1.0)
        
        let rect = CGRect(x: 0.0, y: 0.0, width: size.width, height: size.height)
        
        draw(context, rect)
        
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return image!
    }
    
}
{% endhighlight %}

#### UICollectionView表示クラス

・ViewController.swift
{% highlight Swift %}
extension PlaylistSearchModalView: UICollectionViewDataSource,UICollectionViewDelegate, UICollectionViewDelegateFlowLayout {
    
    //時前に用意したCellのXibファイルを関連づけする
    //refer:https://github.com/sgr-ksmt/BeautifulGridLayout/blob/master/Demo/Demo/SamplePhotoSelectVC.swift
    func registarCollectionCell(){
        
        collectionView.register(UINib(nibName: "GenresCollectionViewCell", bundle: nil), forCellWithReuseIdentifier: "GenresCollectionViewCell")
        
        //ViewControllerをUICollectionViewDataSourceのサブクラスとする設定は、extensionに記述
        collectionView.dataSource = self
        collectionView.delegate = self
        
    }

    //マージン調整
    //http://stackoverflow.com/questions/28325277/how-to-set-cell-spacing-and-uicollectionview-uicollectionviewflowlayout-size-r
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        let itemsPerRow:CGFloat = 2
        let hardCodedPadding:CGFloat = 5
        let itemWidth = (collectionView.bounds.width / itemsPerRow) - hardCodedPadding
        let itemHeight = collectionView.bounds.height/(itemsPerRow * 2)
        return CGSize(width: itemWidth, height: itemHeight)
    }
    
    
    /*
     * UICollectionViewDataSource
     */
    //cellの個数を返す(必須)
    public func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return searchResults.count
    }

    //cellの中身を作成する(必須)。APIで非同期に画像やStringを取得するには、この内部で行う
    //param: indexPath -> 要素を表す
    public func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        
        //Xibで定義したCellと関連づけする
        //dequeueReusableCell → UICollectionView ではあらかじめセルのクラスを登録しておくことで、再利用可能なセルがない場合に新しく作成したものを返してくれるようになっている
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "GenresCollectionViewCell", for: indexPath) as! GenresCollectionViewCell
        let cellImageView: UIImageView = cell.viewWithTag(301) as! UIImageView

        let thumbnailUrl: String? = self.searchResults[indexPath.row].thumbnailUrl

        let url: URL? = URL(string: thumbnailUrl!)
        if url != nil {
            cellImageView.af_setImage(withURL: url!)
        }else{
          //画像が取得できなかった時の処理
          cellImageView.image = UIImage(named: "ic_music_file")
        }
        
        //TableViewのcellをxibファイルで作成した時は、viewWithTagでセルに含まれるアイテムを取得する
       // let titleLabel: UITextField = cell.viewWithTag(302) as! UITextField
        let titleLabel: UILabel = cell.viewWithTag(302) as! UILabel
        titleLabel.text = self.searchResults[indexPath.row].title
        if (cell.isSelected) {
        }
        else
        {
            ///選択れていない時は色を戻す
            cellImageView.image? = (cellImageView.image?.tint(tintColor: UIColor.clear))!
            cell.backgroundColor = deselectedColor
        }
        return cell
    }

    /*
     * UICollectionViewDelegate
     */
    //cellを選択した時に呼ばれる
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        
        let cell = collectionView.cellForItem(at: indexPath)
        if self.collectionView.cellForItem(at: indexPath) == nil {
            return
        }
        
        let selectedCell = self.collectionView.cellForItem(at: indexPath)!
        
        if  selectedCell.isSelected != false{
            
            let cellImageView: UIImageView = cell!.viewWithTag(301) as! UIImageView
            //画像のMaskを変える
            cellImageView.image? = (cellImageView.image?.tint(tintColor: selectedColor))!
            //背景を変える
            cell?.backgroundColor = selectedColor
        }
        
        self.selectedPlaylist = self.searchResults[indexPath.row]

    }
    
    //cell選択解除した時に、カラーマスクを元に戻す
    func collectionView(_ collectionView: UICollectionView, didDeselectItemAt indexPath: IndexPath){
        //let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PlaylistCollectionViewCell", for: indexPath) as! PlaylistCollectionViewCell
         let cell = collectionView.cellForItem(at: indexPath)
        let deselectedCell = self.collectionView.cellForItem(at: indexPath) as? PlaylistCollectionViewCell
        
        if deselectedCell != nil {
            let cellImageView: UIImageView = cell!.viewWithTag(301) as! UIImageView
            //画像のMaskを変える
            cellImageView.image? = (cellImageView.image?.tint(tintColor: deselectedColor))!
            //背景を変える
            cell?.backgroundColor = deselectedColor
           // print("didSelectItemAt2")
        }
        
        self.collectionView.reloadData()
    }
    
    func collectionView(_ collectionView: UICollectionView, shouldSelectItemAt indexPath: IndexPath) -> Bool {
        _ = collectionView.dequeueReusableCell(withReuseIdentifier: "GenresCollectionViewCell", for: indexPath) as! GenresCollectionViewCell
       
      //  print("shouldSelectItemAt")
        return  true
    }
    
    func collectionView(_ collectionView: UICollectionView, shouldDeselectItemAt indexPath: IndexPath) -> Bool {
      //  print("shouldDeselectItemAt")
        return false
    }
}
{% endhighlight %}