# Screen
用來執行背景程式

1. install screen
```
$ apt-get install screen
```
2. 進入screen
```
$ screen
```
3. 離開screen
```
ctrl + A + D
```
4. 查看背景執行程序
```
$ screen -ls
```
5. 關閉背景執行
```
$ screen -X -S 4603.pts-1.3b591b3b4362 quit
```