---
title: webpack升级v4-v5踩坑记
date: 2022-01-22 20:14:36
update: 2022-01-22 20:14:36 
categories: Webpack
---
# 报错1 copy-webpack-plugin

![image-20220118143951437](1.png)

之前写法

```javascript
      new CopyWebpackPlugin([
        {
          from: _src_path + '/assets',
          to: _build_pro_path + '/assets'
        },
      ]),
```

升级后写法

```javascript
      new CopyWebpackPlugin({
        patterns: [
          {
            from: _src_path + '/assets',
            to: _build_pro_path + '/assets'
          },
          {
            from: _src_path + '/iuap-cmdm-fun',
            to: _build_pro_path + '/iuap-cmdm-fun'
          },
        ]
      }),
```

# 报错2 hard-source-webpack-plugin

![image-20220118204432823](2.png)

可以看到是hard-source-webpack-plugin的问题，webpack5内置了缓存策略，所以就不需要使用该插件了。

# 报错3 html-webpack-include-assets-plugin

![image-20220120140015085](3.png)

是因为html-webpack-include-assets-plugin的npm包改名为**html-webpack-tags-plugin**了，重新

```javascript
npm i -D html-webpack-tags-plugin
```

修改包引入方式

```javascript
const HtmlWebpackIncludeAssetsPlugin = require('html-webpack-tags-plugin');
```

# 报错4 less-loader

![image-20220120190613588](4.png)

是less-loader插件报的错, 升级后的版本已经不支持旧版本的写法。

旧版本写法

```javascript
          {
              loader: "less-loader",
              options: {
                  javascriptEnabled: true
              }
          }
```

新版本写法

```javascript
          {
            loader: "less-loader",
            options: {
              lessOptions: {
                javascriptEnabled: true
              }
            }
          }
```

# 报错5 webpack5不自动引入Polyfills

![image-20220120192356906](6.png)

报错原因：

v4以前附带了许多[node](https://so.csdn.net/so/search?q=node&spm=1001.2101.3001.7020).js核心模块的`polyfill`，在构建时给 bundle附加了庞大的polyfills，在大部分情况下，polyfills并不是必须。

所以现在v5将要停止这一切，在模块的应用中不再自动引入`Polyfills`，明显的减小了打包体积。

如果我们要使用这个polyfill我们就在resolve.fallback中引入，如下

```javascript
resolve: {
      fallback: {
        util: require.resolve('util/'),
      },
}
```

如果确认这个polyfill我们没有使用，可以不引入，如下

```javascript
  resolve: {
    fallback: {
      util: false,
    },
  },
```

```

![image-20220122110507900](10.png)

```javascript
    fallback: {
      util: false,
      crypto: false,
      path: false
    },
```



# 报错6 资源模块

![image-20220120195255483](7.png)

未升级版本时写法

```javascript
      {
        test: /\.(eot|ttf|woff|woff2|svg|svgz)(\?.+)?$/,
        use: [{
          loader: "file-loader",
          options: {
            name: '[name].[ext]',
            outputPath: 'fonts',
          }
        }]
      }
```

通过查看webpack5官网后，发现以下一段话

> 当在 webpack 5 中使用旧的 assets loader（如 `file-loader`/`url-loader`/`raw-loader` 等）和 asset 模块时，你可能想停止当前 asset 模块的处理，并再次启动处理，这可能会导致 asset 重复，你可以通过将 asset 模块的类型设置为 `'javascript/auto'` 来解决。

解决方式

1. 添加type: 'javascript/auto'

```javascript
      {
        test: /\.(eot|ttf|woff|woff2|svg|svgz)(\?.+)?$/,
        use: [{
          loader: "file-loader",
          options: {
            esModule: false, // 不加的话会有这种情况 img属性src="[object Module]"
            name: '[name].[ext]',
            outputPath: 'fonts',
          }
        }],
          type: 'javascript/auto'
      }
```

2. 使用资源模块类型

   1. 使用方式

   > - `asset/resource` 发送一个单独的文件并导出 URL。之前通过使用 `file-loader` 实现。
   > - `asset/inline` 导出一个资源的 data URI。之前通过使用 `url-loader` 实现。
   > - `asset/source` 导出资源的源代码。之前通过使用 `raw-loader` 实现。
   > - `asset` 在导出一个 data URI 和发送一个单独的文件之间自动选择。之前通过使用 `url-loader`，并且配置资源体积限制实现。

   2. 这里通过asset实现

   ```javascript
         {
           test: /\.(eot|ttf|woff|woff2|svg|svgz)(\?.+)?$/,
           type: 'asset',
           generator: {
             filename: "fonts/[name].[ext]"
             
           },
           parser: {
             dataUrlCondition: {
               limit: 8196
             }
           }
         }
   ```

# 报错7 html-webpack-plugin

```javascript
ERROR in   Error: The loader "/Users/chenlili/yonyou/versionTotal/develop-cll-test-webpack/iuap_cmdm_front/node_modules/html-webpack-plugin/lib/loader.js!/Users/chenlili/yonyou/versionTotal/develop-cll-test-webpac  k/iuap_cmdm_front/src/pages/model-management/flow-model/index.html" didn't return html.
```

![image-20220122101322313](11.png)

发现这个问题的时候直接百度了，看到有哥们说是html-webpack-plugin内置代码的问题，他修改了源码，但是我觉得这种处理方式不太完美，就打开了html-webpack-plugin在github的issues。确实有人遇到了同样的问题，给出的解决方式是升级html-webpack-plugin版本，于是我升级到了5.5.0。

```javascript
npm install html-webpack-plugin@5.5.0 -D
```

至此，单模块的打包可算是构建成功了。

![image-20220122102521491](12.png)



>  接下来尝试整个项目打包，看看会遇到什么问题？

开始了新一轮的WARNING和Error！！！



# 报错8 react-dnd

![image-20220122105143120](13.png)

应该是react-dnd的问题，升级react-dnd @9.4.0 -> 最新版本@14.5.0

# 报错9 react-dnd-html5-backend

![image-20220122110741387](/14.png)

盲猜react-dnd-html5-backend问题，升级react-dnd-html5-backend最新版本。

出现一个warning

![image-20220122111828816](15.png)

源代码引入方式

![image-20220122111849687](16.png)

更改为

```javascript
import {HTML5Backend} from 'react-dnd-html5-backend';
```

