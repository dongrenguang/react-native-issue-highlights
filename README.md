# react-native-issue-highlights
React Native问题集锦，记录我在使用React Native（主要是Android）过程中遇到的问题。

## 1. ReferenceError: Can't find variable: __fbBatchedBridge
<div align="center"><img src="./images/red.png"  width="300px"  align="center" /></div>

使用初期经常会遇到的问题，一在模拟器（或真机）上跑，就出现满江红。试了[StackOverflow上的解决办法](http://stackoverflow.com/questions/34500020/referenceerror-cant-find-variable-fbbatchedbridge)，依然不行。愤懑之下重新装了一个模拟器就好了。。。不明觉厉。

## 2. 实现圆形切图
<div align="center"><img src="./images/circle.png"  align="center" /></div>

```javascript
<Image
    style={{height: 100, width: 100, borderRadius: (100 / 2) }}
    source={**} />
```
一定不能加`resizeMode`，否则怎么切图都不对。
PS：在其他情况下也是，慎用`resizeMode`。

## 3. Navigator中Scene的滑动问题
<div align="center"><img src="./images/home-scene.png"  width="300px" align="center" /></div>

在使用Navigator的时候，滑动当前的Scene可能会导致当前的Scene的位置移动，有时这并不是我们想要的。比如在最顶层的Navigator，我们不希望主页Scene的位置随着手的滑动而“漂移”，
```javascript
<Navigator
    initialRoute={initialRoute}
    renderScene={this._renderScene}
/>
```
其实只需要不加`configureScene`属性就好了，主页Scene坚如磐石。而且在主页Scene与其他Scene之间切换的时候，会有一个默认的、平滑的动画。

## 4. TypeError: Network request failed
在本地也启动了一个服务器（Tomcat，基于JavaEE，比如：localhost:8080），作为应用的后台，提供数据操作等。因为App在模拟器上跑的时候在本地也启动了一个服务器（为了便于调试，比如：localhost：8081）。在使用`fetch`方法进行访问后台操作时，报`TypeError: Network request failed`错。[Github](https://github.com/facebook/react-native/issues/5584)或[StackOverflow](http://stackoverflow.com/questions/34570193/react-native-post-request-via-fetch-throws-network-request-failed)上的答案都无法解决这个问题。后来我把`localhost`改成了`本机IP地址`，问题就解决了。
```javascript
async _fetchData() {
    const responseJson = await fetch("http://192.168.56.1:8080/ScholarHome/News/SearchNews", {
        method: "POST",
        headers: {
          "Accept": "application/json",
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
            "channelId": "2",
            "pageIndex": "0",
            "pageSize": "10",
        })
    })
    .catch((error) => {
        console.warn(error);
    });

    if (responseJson.ok) {
        const dataJson = await responseJson.json().catch((error) => {
            console.warn(error);
        });

        console.log(dataJson.data.topNews);
    }
}
```

## 5. 删除[ListView](https://facebook.github.io/react-native/docs/listview.html#content)子元素问题
删除`dataSource`中的某个值，React是这么做的：删掉`ListView`中的最后一个物理子元素，然后更新前面的物理子元素的`rowData`值。

## 6. 在父组件中调用子组件实例中的方法
在创建的子组件上应用`ref`属性，就可以在父组件中直接拿到子组件的实例对象了。详见[Refs to Components](http://facebook.github.io/react/docs/more-about-refs.html)

## 7. 在子组件中调用父组件中的方法
在父组件中实例化子组件的时候，只要将父组件的方法（bind了this）作为一个`property`传递给子组件就好了。

## 8. [ViewPagerAndroid](https://facebook.github.io/react-native/docs/viewpagerandroid.html#content)组件更新子`View`时的bug
问题描述详见：[ViewPagerAndroid returns empty view when new source data added](https://github.com/facebook/react-native/issues/4775)

搞了三四天才解决掉的神bug！！是`React Native`的`ViewPagerAndroid`组件的一个bug，望Facebook尽快解决（截至@0.22版）。网上找到了三种解决办法:
- 给`ViewPagerAndroid`添加`key`属性，在适当的时候（比如在删除某一个子View之后）更新这个`key`值（也就是让`React Native`彻底重绘此`ViewPagerAndroid`组件）；
- 使用第三方的组件[react-native-viewpager](https://github.com/race604/react-native-viewpager)取代`ViewPagerAndroid`；
- 期待Facebook尽快解决此问题，为Android平台的`ScrollView`添加`pagingEnabled`属性。

因为牵扯到的组件、逻辑太过复杂，只尝试了第一种方法，亲测可行，只不过是以牺牲性能为代价的（因为`key`变了，需要重绘整个组件），而且整个组建的变化看上去也不那么连贯（动画）；第二种方法方法据说性能较低；第三种方法有点空想社会主义。

## 9. 在[Navigator](https://facebook.github.io/react-native/docs/navigator.html#content)中调用`Scene`实例中的方法
在顶层`Navigator`中，当从一个`Scene`（B）结束跳到之前的一个`Scene`（A）的时候，可能需要刷新`Scene A`（受在`Scene B`中的操作结果的影响）。只需要在顶层`Navigator`中设置`onWillFocus`属性（还有一个`onDidFocus`属性）。
```javascript
import BottomTabs from "./scenes/tabBar/BottomTabs";
......

export default class Application extends Component {
    ......

    render() {
        const initialRoute = {
            scene: "bottomTabBar",
            view: BottomTabs,
            handleWillFocus: BottomTabs.handleWillFocus,
        };

        return (
            <Navigator
                initialRoute={initialRoute}
                renderScene={this._renderScene.bind(this)}
                onWillFocus={(route) => {
                    if (route.handleWillFocus && (typeof route.handleWillFocus === "function")) {
                        route.handleWillFocus();
                    }
                }}
            />
        );
    }

}
```
这里，`BottomTabs`的`handleWillFocus`是属于`BottomTabs`的类方法，而不是实例方法，具体的`BottomTabs`中的方法如下所示：
```javascript
let bottomTabsSingleton = null;   // Someting like "super singleton".
export default class BottomTabs extends Component {
    constructor(props) {
        super(props);
        ......
    }

    componentWillMount() {
        bottomTabsSingleton = this;   
    }

    componentWillUnmount() {
        bottomTabsSingleton = null;     
    }

    updateSubScenes() {
        ......
    }

}

BottomTabs.handleWillFocus = () => {
    if (bottomTabsSingleton) {
        bottomTabsSingleton.updateSubScenes();
    }
};
```
这里的`handleWillFocus()`，和`bottomTabsSingleton`是属于`BottomTabs`类的，而不是某一个实例对象的。`bottomTabsSingleton`某种程度上有点类似于单例模式，不过它总是指向最近创建的那个`BottomTabs`实例对象。这种方法也只适用于比较复杂的、只需要创建一次的组件的情况。

## 10. 所谓的单向数据流
一些前端框架（比如：AngularJS）的数据绑定是双向的，时常会造成状态混乱，难以控制。Facebook就是为了避免双向数据绑定所以才开发了所谓单向数据流的React —— 父组件向子组件传递property，在组件中通过setState()方法来控制视图更新。然而今天发现，不通过调用setState()方法也能更新视图：当组件的state是一个对象（或数组）的时候，直接改变这个state的值，不需要使用setState()方法也会造成状态更新。
```javascript
const name = ...
const content = ...
......

const dataSource = this.state.dataSource;
for (let rowData of dataSource) {
    if (rowData["name"] === name) {
        rowData["content"] = content;
    }
}
```
上面的代码中，用dataSource变量取得state值——dataSource（是一个数组，里面的元素是对象），直接修改dataSource数组元素中的值，然后直接导致视图更新。所以当你的state值是一个对象（数组）的时候，就要当心了。如果想要避免此类问题，在取得dataSource值的时候，需要用到深度对象拷贝(比如用JSON转化的方法)，代码如下：
```javascript
const name = ...
const content = ...
......

const dataSource = JSON.parse(JSON.stringify(this.state.dataSource));
for (let rowData of dataSource) {
    if (rowData["name"] === name) {
        rowData["content"] = content;
    }
}
......

this.setState({
    dataSource,
});
```

## 11. `Touchable`系列的组件的`margin`与`padding`属性
`margin`加在外面的`Touchable**`上，不要加在里层的`View`上，不然造成点击`margin`位置的时候也会相应点击事件；`padding`可以加在里层的`View上`。

## 12. `Image`大小的自适应
<div align="center">
    <img src="./images/share-box.png"  width="150px"  align="center" />
</div>
如上所示的组件：上部是一张图片，下部是文字。如果想让图片占据父组件高度的3/4，下部文字占余下的1/4，那么你也许会想直接给`<Image>`组件样式的`flex`属相设置为3，下部`<Text>`组件的样式的`flex`属性设置为1，然而大概在`React Native` 0.30版以后，这种方法就无法达到预期的效果。正确的做法是在<Image>组件外层再嵌套一个`<View>`组件，其`flex`值为3，而嵌套在`<View>`中的<Image>的`flex`值设为-1来达到自适应的效果。

具体代码示例：
```javascript
<TouchableOpacity style={styles.shareBox} onPress={handlePress}>
    <View style={styles.shareIconWrapper}>
        <Image
            style={styles.shareIcon}
            resizeMode={'contain'}
            source={iconSource}
        />
    </View>
    <Text numberOfLines={1} style={styles.shareIconText}>
        {title}
    </Text>
</TouchableOpacity>

......

const styles = StyleSheet.create({
    shareBox: {
        flex: 1,
        alignSelf: 'stretch',
        alignItems: 'center',
    },
    shareIconWrapper: {
        flex: 3,
    },
    shareIcon: {
        flex: -1,
    },
    shareIconText: {
        flex: 1,
        fontSize: 13,
        color: 'rgba(150, 150, 150, 1)',
        textAlign: 'center',
        textAlignVertical: 'top',   // Android only.
    },
});
```
