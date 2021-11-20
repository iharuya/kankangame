# カンカンするゲーム
よくみるボール反射ゲーム。
Brick Crusherとかもっといい名前があっただろうけど、そこについてはノータッチで。

## Function
スタート（Canvasコンテキストをクリック）すると左右に揺れる矢印が現れる。  
もう一度コンテキスト（または矢印の上）をクリックすると、その方向にボールが発射される。

ボールはボックス（数字を持ったブロック）にあたると反射する。  
その際、ボックスの数字が減り、0になると消える。

ボールは下部の赤い部分（DeadZone）に接触すると描画の更新が止まり、ゲーム終了。  
終了後の結果アラートをOKすると、初期画面に戻る。

## 仕組み（概略）
### start(initGame())
Ball、Box、DeadZone各クラスを用意し、
initgame()でインスタンスを作成、この時、Boxは複数作成され、boxes配列に格納される。

矢印がスイングしている状態でクリックしたら、Canvas全体の描画を更新し続けるsetIntervalが発火。

### ゲーム描画(update())
このsetIntervalがコールバックするupdate関数では
- コンテクストをリセット(clearRect)
- 判定
  - ボールが壁に当たっているか(Ballのインスタンスメソッド)
  - boxes配列をsomeして、各ボックスに対して当たり判定
    - ここでは横の面or縦の面orいずれでもないを領域を用いて判定
    - もし、衝突判定が出たら**return trueでループを抜ける**(一度のupdate()でのボックス反射は一度だけ)
    - ※衝突判定はバグを防ぐために次のアップデート後の位置に対しても判定しています（下参照）
- 判定に基づく操作
  - ボール
    - 左右の壁に当たったら速度vx *= -1でx軸方向の速度を反転
    - 上の壁の場合、vy *= -1
    - デットゾーン（下部に入ったら）finishGame()
  - ボックス
    - 左右面に接触->ボールのvx *= -1 
    - 上下面 -> vy *= -1
    - いづれも、metBox()メソッドを発火でスコア+1、ボックスの数字-1
    - もし、数字-1して0になる時、boxesからインスタンス変数idを参照してspliceで削除
    - 衝突の後数十回更新する間は、そのボックスの背景を赤にする
- 描画
  - Ball, Boxのdrawメソッドで描画
  - デットゾーンを描画（drawメソッドで背景色をボール位置に基づき判定）

### finish(finishGame())
- clearIntervalでupdateの繰り返しをストップ
- alert()で結果スコア表示
- initGame()

## ボックスとボールの当たり判定
ボールがボックスの斜めから進入すると、稀にボックス辺付近で反射が繰り返し起こるバグが起こる。  
これは、本来上下方向から進入したボールが、判定されるフレームでは左右領域にあることで発生する。  
(and vice versa)  
したがって、衝突判定が出て、左右から進入したという判定があったとするなら、
- その判定後の速度を別の変数に格納
- その速度を使って、次のフレームにおけるボールの座標を導出
- 次のフレームの場所も左右判定がtrueになるような場合、これはおかしい
- その場合、**vx *= -1ではなく、vy *= -1**として繰り返し反射を回避する
