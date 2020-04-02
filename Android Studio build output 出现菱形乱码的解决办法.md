# Android Studio build output 出现菱形乱码的解决办法

解决办法：

1. 打开Android Studio，但是不要加载工程
2. 打开Configure —> Edit Custom VM Options
3. 添加如下内容
	>-Dfile.encoding=UTF-8
4. 重启Android Studio

