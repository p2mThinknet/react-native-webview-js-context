# react-native-webview-js-context [![npm version](https://badge.fury.io/js/react-native-webview-js-context.svg)](http://badge.fury.io/js/react-native-webview-js-context)

Interactive JavaScript between a UIWebView and React Native.  


```javascript
import React, {Component} from 'react';
import {
  AppRegistry,
  StyleSheet,
  NativeModules,
  Text,
  View,
  Platform,
  TouchableHighlight,
  Image
} from 'react-native';

import ReactDOMServer from 'react-dom/server';


const ROWCOUNT_JS = `
         //1 in = 96 px
         //页边距左右各给2.54cm = (2.54 * 0.393700787) * 96(px)
         //A4纸张宽度 (8.3 in * 96)px
         function setDivWidthForA4(divClass) {
            var divComponent = document.getElementsByClassName(divClass)[0];
            var divWidth = (8.3 - 2 * 2.54 * 0.393700787) * 96;
            divComponent.style.width = divWidth + 'px';
         };

         /*function getParagraphRowCount(paragraph) {

         }*/

         function getRowCountFromDiv(divClass, paragraphSpace, rowHeight) {
            var divComponent = document.getElementsByClassName(divClass)[0];        
            setDivWidthForA4(divClass);
            var divWidth = divComponent.offsetWidth;
            var divHeight = divComponent.offsetHeight;
            var paragraphCount = divComponent.getElementsByTagName("p").length;
            var rowCount = parseInt((divHeight - (paragraphCount - 1) * paragraphSpace) / (rowHeight) + 0.5);
            return rowCount;
         };
         var rows = document.getElementsByClassName("scope")[0].getElementsByTagName("p").length;
         var num = getRowCountFromDiv("scope", 28, 20, 20);
         resolve(num + "行") /* <--- resolve() is called by RNWebViewJSContext */`;

import WebViewJSContext from 'react-native-webview-js-context';

class Greeting extends React.Component {
  constructor() {
    super();
  }

  render() {
    return (
        <div className="scope">
          <p>
            <span >
                <span>时间</span>
            </span>
          <span className="text_u">2016 </span>
          <span >年</span>
          <span className="text_u"><span >07 </span></span>
          <span ><span>月</span></span>
          <span className="text_u"><span >01 </span></span>
          <span ><span>日</span></span>
          <span className="text_u"><span >11 </span></span>
          <span ><span>时</span></span>
          <span className="text_u"><span >15 </span></span>
          <span ><span>分至</span></span>
          <span className="text_u"><span >11 </span></span>
          <span ><span>时</span></span>
          <span className="text_u"><span >30 </span></span>
          <span ><span>分</span>
                &nbsp;&nbsp;&nbsp;&nbsp;<span>第</span> &nbsp;1次询问</span>        
        </p>
        <p>
          <span >地点</span>
          <span className="text_u"><span >&nbsp;G6京藏高速公路叶盛收费站&nbsp;</span>
            </span>       
        </p>
        <p>
          <span ><span>询问人</span></span>
          <span className="text_u">
                  <span >&nbsp;<span>管仲</span>&nbsp;<span>公孙鞅</span>&nbsp;</span>
              </span>
          <span ><span>记录人</span></span>
          <span className="text_u"><span >
                  &nbsp;
                  <span>韩非</span> &nbsp;</span>
              </span>       
        </p>
      </div>
    );
  }
}

export default class RNPrintExample extends Component {

  /*
  divComponent: Rect组件
  paragraphSpace: 段间距(px)
  rowHeight: 行高(px)
  */
  getRowCountFromDiv(divComponent, paragraphSpace, rowHeight) {
    let divHTMLStr = `<html>
                        <head>                         
                          <script type="text/javascript">        
                              window.onload = resolve;
                          </script>
                        </head>
                        <body>`;
    divHTMLStr += ReactDOMServer.renderToString(React.createElement(divComponent));
    divHTMLStr += `</body>
                    </html>`;
    WebViewJSContext
      .createWithHTML(divHTMLStr)
      .then(context => {
        this.ctx = context;
        this.caculateRowCount();
      });
  }

  componentWillUnmount() {
    this.ctx && this
      .ctx
      .destroy();
  }

  render() {
    return (
      <View>
        <TouchableHighlight
          onPress={() => {
          this.getRowCountFromDiv(Greeting, 66, 20);
        }}>
          <Text>计算行数</Text>
        </TouchableHighlight>
      </View>
    )
  }

  async caculateRowCount() {
    var rowCount = await this.ctx.evaluateScript(ROWCOUNT_JS);
    alert(rowCount);
    this.ctx && this
      .ctx
      .destroy();
  }
}

AppRegistry.registerComponent('RNPrintExample', () => RNPrintExample);
```

## Usage

First you need to install react-native-webview-js-context:

```javascript
npm install react-native-webview-js-context --save
```


## `iOS`

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-webview-js-context` ➜ `ios` and add `RNWebViewJSContext.xcodeproj`
3. In XCode, in the project navigator, select your project. Add `libRNWebViewJSContext.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
4. Run your project (`Cmd+R`)

### `Android`

* `android/settings.gradle`

```gradle
...
include ':react-native-webview-js-context'
project(':react-native-webview-js-context').projectDir = new File(settingsDir, '../node_modules/react-native-webview-js-context/android')
```
* `android/app/build.gradle`

```gradle
dependencies {
	...
	compile project(':react-native-webview-js-context')
}
```

* register module (in MainApplication.java)

```java
...

import com.shaynesweeney.react_native_webview_js_context.RNWebViewJSContextPackage; // <--- IMPORT

public class MainActivity extends Activity implements DefaultHardwareBackBtnHandler {
	...

  @Override
  protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
          new RNPrintPackage(),
          new RNHTMLtoPDFPackage(),
          new RNWebViewJSContextPackage()   //<----- add here
    );
  }
}
```
