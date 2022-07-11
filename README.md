
Copyright &copy; 2022 douemon. All rights reserved.

See the end of this file for further copyright and license information.

Python project samples for the fresh year exercise

# インストール

[douemon/python_sample](https://github.com/douemon/python_sample.git) の code からダウンロード

# 使用法

> python final project.py

# コード解説


## ブロック崩し
引用したモノにいろいろ付け加えてみた.


### 基本設定
tkinter,sys,random,mathをimport.

ウィンドウのサイズ決定

```Python
import tkinter as tk
import sys
import random
import math

#基本設定
window_width = 600
window_height = 480
window_center_x = window_width/2
window_center_y = window_height/2
tick = 40    # ティック数（ミリ秒）

root = tk.Tk()
root.title(u"ブロック崩し")
root.geometry("600x480")
cv = tk.Canvas(root, width = window_width, height = window_height)
cv.pack()
```


### ボールのクラス

変更点
- 引用したものはボールが１つだけだったので増やそうとしたが、うまく出来なかったのでクラスを２つ作った.
- パドルで反射すると少し早くなる.
- また、ボールの生成場所をランダムにしたので、飽きにくいかも.その代わりブロックに当たらなくなる可能性あり.
- 1つ目のボールが画面下に行ったときに、最初のままだとどこかで反射して帰ってきたので、画面外に行くとdx,dyが0になるようにした.


#### Ball1
１つ目のボールはmathを使って移動方向を決めた.
スピード固定で角度を変える事が出来る.
```Python
class Ball1:
    x = random.randint(10, 592)
    y = random.randint(100,300)
    w = 10
    speed = 3
    angle = math.radians(60)
    dx = speed * math.cos(angle)
    dy = speed * math.sin(angle)
    color = "red"
    def draw(self):
        cv.create_oval(self.x-self.w, self.y-self.w, self.x+self.w, self.y+self.w, fill = self.color, tag = "ball")
        cv.pack

    def move(self):
       
        #x,y座標を動かす
        self.x += self.dx
        self.y += self.dy
        
    
        #対壁
        if self.x - self.w < 0 or self.x + self.w > window_width :
            self.dx *= -1
        if self.y - self.w < 0 :
            self.dy *= -1
        if self.y + self.w > window_height +17:
            self.dx = self.dy = 0


        #対パドル
        if self.y + self.w > paddle.y - paddle.wy and ball1.x > paddle.x-paddle.wx and ball1.x < paddle.x+paddle.wx:
            self.dy *= -1.01
            self.dx *=  1.01
        
    def delete(self):
        cv.delete("ball")
```
#### Ball2
移動量が固定なので角度を変えようとすると早くなる.

後はBall1と一緒.
```Python
class Ball2:
    x = random.randint(8, 590)
    y = random.randint(100, 300)
    w = 10
    dx = dy = 2
    color ="red"
        
    def draw(self):
        cv.create_oval(self.x-self.w, self.y-self.w, self.x+self.w, self.y+self.w, fill = self.color, tag = "ball")
        cv.pack

    def move(self):
       
        #x,y座標を動かす
        self.x += self.dx
        self.y += self.dy
        
    
        #対壁
        if self.x - self.w < 0 or self.x + self.w > window_width:
            self.dx *= -1
        if self.y - self.w < 0 :
            self.dy *= -1
        if self.y + self.w > window_height +17:
            self.dx = self.dy =  0
        

        #対パドル
        if self.y + self.w > paddle.y - paddle.wy and ball2.x > paddle.x-paddle.wx and ball2.x < paddle.x+paddle.wx :
            self.dy *= -1.01 
            self.dx *=  1.01  # 少しだけ早くなる
        

    def delete(self):
        cv.delete("ball")
```

### パドルのクラス
ボールの数が増えたので、パドルの幅も大きくした.

最初はカーソルで動かせるようにしようと頑張ったが断念.
```Python
class Paddle:
    x = window_center_x   #初期値ｙ
    y = window_height -30 #初期値ｘ
    wx= 120               #幅ｘ
    wy= 8                 #幅ｙ
    dx= 6                 #移動量
    color = "blue"       
    def draw(self):
        cv.create_rectangle(self.x - self.wx, self.y - self.wy, self.x + self.wx, self.y +self.wy, fill = self.color, tag = "paddle" )

    def right(self,event):
        cv.delete("paddle")
        self.x += self.dx
        self.draw()
    def left(self,event):
        cv.delete("paddle")
        self.x -= self.dx
        self.draw()

    def move(self):
        root.bind("<Right>", self.right)
        root.bind("<Left>", self.left)
```

### ブロックのクラス
ブロックの幅を変え数を増やしてゲームがすぐに終わらないようにした.
また、ボールのクラスが2つになったのでコピーした.

ブロックを消すとスピードアップ

```Python
class Block:
    w_x = 50  #幅ｘ
    w_y = 30  #幅ｙ
    global dy, score #衝突時にボールのクラスの移動量、スコアを変更したいので、グローバルにする
    
    #ブロックのスイッチ　1:on,0:off
    block_list =[[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1], #j=0,i=0~23
                 [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1], #j=1,i=0~23
                 [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]] #j=2,i=0~23
    
    def draw(self):
        for i in range(12):
            for j in range(3):
                cv.create_rectangle(i*self.w_x, j*self.w_y, (i+1)*self.w_x, (j+1)*self.w_y, fill = "orange", tag = "block"+str(j)+str(i))
            
    def reflect(self):
        for i in range(24):
            for j in range(3):
                #ボールが上から反射
                if (ball1.y-ball1.w <= (j+1)*self.w_y  #ボールがブロックよりも下
                    and i*self.w_x < ball1.x < (i+1)*self.w_x #ボールがブロックの左右に挟まれている
                    and self.block_list[j][i] == 1): #スイッチが1
                        ball1.dy *= -1.2 #反射させる
                        ball1.dx *=  1.2
                        cv.delete("block"+str(j)+str(i)) #ブロックの描画を消す
                        self.block_list[j][i] = 0 #スイッチを切る
                        score.score += 1  #スコアの加点
                        score.delete()   #スコアを更新
                        score.draw()     

                #ボールが上から反射
                if (ball2.y-ball2.w < (j+1)*self.w_y  #ボールがブロックよりも下
                    and i*self.w_x < ball2.x < (i+1)*self.w_x #ボールがブロックの左右に挟まれている
                    and self.block_list[j][i] == 1): #スイッチが1
                        ball2.dy *= -1.2 #反射させる
                        ball2.dx *=  1.2
                        cv.delete("block"+str(j)+str(i)) #ブロックの描画を消す
                        self.block_list[j][i] = 0 #スイッチを切る
                        score.score += 1  #スコアの加点
                        score.delete()   #スコアを更新
                        score.draw()     
```

### スコアのクラス
スコアは1ずつ増えていくはずだけど、ボールが高速でブロックを消したとき何故か時々おかしくなってしまう.
```Python
class Score():
    score = 0  #初期値
    def draw(self):
        cv.create_text(window_width - 50, 50, text = "Score = " + str(self.score), font = ('FixedSys', 16), tag = "score")
    def delete(self):
        cv.delete("score")

```
### ゲーム終了

#### ゲームオーバー

ボールがすべて画面外に行くとゲームオーバー
```Python 
def gameover():
    global w
    if ball1.y + ball1.w > window_height and ball2.y + ball2.w > window_height:
        cv.delete("all")
        #cv.delete("paddle")
        #cv.delete("ball")
        score.draw()
        cv.create_text(window_center_x, window_center_y, text = "GAME OVER", font = ('FixedSys', 40))
        ball1.w = ball2.w = 0
  ```


#### ゲームクリア
スコアが36，つまりブロックをすべて消すとクリア
```Python
def gameclear():
    global w
    if score.score == 36:
        cv.delete("paddle")
        cv.delete("ball")
        cv.create_text(window_center_x, window_center_y, text = "GAME CLEAR", font = ('FixedSys', 40))
        ball1.w = ball2.w = 0
```
## ゲーム生成
```Python
#インスタンス生成
paddle = Paddle()
ball1  = Ball1()
ball2  = Ball2() 
block  = Block()
score  = Score()

#初期描画   
ball1.draw()
ball2.draw()
paddle.draw()
block.draw()
score.draw()

#ゲームのメインループ
def gameloop():
    ball1.delete()        #ボールを消す
    ball2.delete()
    ball1.move()          #ボールを動かす
    ball2.move()
    paddle.move()        #パドルを動かす
    block.reflect()      #ボールを反射させ、ブロックを消す
    ball1.draw()          #ボールを描く
    ball2.draw()
    gameover()           #表示
    gameclear()          #表示
    root.after(tick, gameloop)  #50ミリ秒経過後、ループの最初に戻る

#メインの実行部分
gameloop()
root.mainloop() #画面を表示
```

## 引用元
[Pythonのtkinterでブロック崩しを作ってみた。](https://qiita.com/Yt330110713/items/ed6beb74c02515b35248)

# Copyright and License Information

This project is licensed under the terms of the MIT license.