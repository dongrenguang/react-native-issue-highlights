# react-native-issue-highlights
React Native问题集锦，记录我在使用React Native（主要是Android）过程中遇到的问题。

## ReferenceError: Can't find variable: __fbBatchedBridge
![](./images/red.png =100x)

使用初期经常会遇到的问题，一在模拟器（或真机）上跑，就出现满江红。试了[StackOverflow上的解决办法](http://stackoverflow.com/questions/34500020/referenceerror-cant-find-variable-fbbatchedbridge)，依然不行。愤懑之下重新装了一个模拟器就好了。。。不明觉厉。

## 实现圆形切图
![](./images/circle.png)

```javascript
    <Image 
        style={{height: 100, width: 100, borderRadius: (100 / 2) }}
        source={**} />
```
一定不能加`resizeMode`，否则怎么切图都不对。

## Navigator中Scene的滑动问题
![](./images/home-scene.png)

在使用Navigator的时候，滑动当前的Scene可能会导致当前的Scene的位置移动，有时这并不是我们想要的。比如在最顶层的Navigator，我们不希望主页Scene的位置随着手的滑动而“漂移”，
```javascript
    <Navigator
        initialRoute={initialRoute}
        renderScene={this._renderScene}
    />
```
其实只需要不加`configureScene`属性就好了，主页Scene坚如磐石。而且在主页Scene与其他Scene之间切换的时候，后有一个默认的、平滑的动画。
