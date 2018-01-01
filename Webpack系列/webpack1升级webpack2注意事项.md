webpack1已结不再维护了，官方在主推webpack2，原先使用webpack1的用户，想要使用v2，只需要做几个配置小改动即可。

我们只关注常用的配置选项，不常用的不做解释。

1、resolve配置

以前你可能这么写：

    resolve: {
		extensions: ['', '.jsx', '.js', '.json'],
		modulesDirectories: ['node_modules', 'src'],
		alias: {
			actions: __dirname + `/src/actions`,
			components: __dirname + `/src/components`,
			containers: __dirname + `/src/containers`,
			reducers: __dirname + `/src/reducers`,
			store: __dirname + `/src/store`
		}
	}

现在你该这么写：取消了空字符串

    resolve: {
        extensions: ['.js', '.jsx', '.less', '.scss', '.css'],
        modules: [
          path.resolve(__dirname, 'node_modules'),
          path.join(__dirname, './src')
        ]
      }

2、module配置

以前你可能这么写：
module: {
		loaders: [{
			test: /\.(js|jsx)$/,
			loaders: ['react-hot', 'babel-loader'],
			exclude: /node_modules/
		}, {
			test:   /\.(less|css)$/,
			loader: "style-loader!css-loader!less!postcss-loader"
		}, {
			test: /\.(png|jpg|gif|svg)$/,
			loader: 'file?name=[md5:hash:base64:10].[ext]'
		}, {
			test: /\.json$/,
			loader: 'json'
		}, {
			test: /\.md$/,
			loader: 'file?name=[name].[ext]'
		}]
	}

现在你该这么写：

a、外层loaders改为rules

b、内层loader改为use

c、所有插件必须加上-loader，不再允许缩写，所以react-hot不需要再配置。

d、不再支持使用！连接插件，请使用数组形式。

e、json-loader已经被移除，不需要手动添加，webpack会帮你处理好这些事情。

    module: {
          rules: [{
              test: /\.(js|jsx)$/,
              use: ['babel-loader'],
              exclude: /node_modules/,
              include: path.join(__dirname, 'src')
          }, {
              test: /\.(less|css)$/,
              use: ["style-loader", "css-loader", "less-loader", "postcss-loader"]
          }, {
              test: /\.(png|jpg|gif|md)$/,
              use: ['file-loader?limit=10000&name=[md5:hash:base64:10].[ext]']
          }, {
              test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
              use: ['url-loader?limit=10000&mimetype=image/svg+xml']
          }],
      }
    };
    
3、plugins配置

以前你可能这么写：
    new webpack.optimize.OccurenceOrderPlugin(),
	new webpack.HotModuleReplacementPlugin(),
	new webpack.NoErrorsPlugin(),
    new webpack.optimize.UglifyJsPlugin()
    
现在你该这么写：

a、移除了OccurenceOrderPlugin 和 NoErrorsPlugin

b、更多plugins配置请参考 https://webpack.js.org

    new webpack.HotModuleReplacementPlugin(),
    new webpack.optimize.UglifyJsPlugin()

==============================================================

以上几个常用配置变化的比较明显，没有修改的配置会报错导致webpack无法使用，请注意遵守webpack2规则。

最后贴上一份webpack.config.js基础配置，查看该配置完整项目请点击：[react-webpack2][1]

    var path = require('path');
    var webpack = require('webpack');
    var autoprefixer = require('autoprefixer');
    var precss = require('precss');
    
    //判断当前运行环境是开发模式还是生产模式
    const nodeEnv = process.env.NODE_ENV || 'development';
    const isPro = nodeEnv === 'production';
    
    console.log("当前运行环境：", isPro)
    
    var plugins = []
    if (isPro) {
      plugins.push(
          new webpack.optimize.UglifyJsPlugin({
              compress: {
                  warnings: false
              }
          }),
          new webpack.DefinePlugin({
              'process.env':{
                  'NODE_ENV': JSON.stringify(nodeEnv)
              }
          })
      )
    } else {
      plugins.push(
          new webpack.DefinePlugin({
              'process.env':{
                  'NODE_ENV': JSON.stringify(nodeEnv)
              },
              BASE_URL: JSON.stringify('http://localhost:9009'),
          }),
          // new webpack.optimize.OccurenceOrderPlugin(),
          new webpack.HotModuleReplacementPlugin()
          // new webpack.NoErrorsPlugin()
      )
    }
    
    module.exports = {
      devtool: false,
      entry: {
        app: [
          'webpack-hot-middleware/client?path=http://localhost:3011/__webpack_hmr&reload=true&noInfo=false&quiet=false',
          'babel-polyfill',
          './src/index'
        ]
      },
      output: {
        filename: '[name].js',
        path: path.join(__dirname, 'build'),
        publicPath: 'http://localhost:3011/build/',
        chunkFilename: '[name].js'
      },
      // BASE_URL是全局的api接口访问地址
      plugins,
      // alias是配置全局的路径入口名称，只要涉及到下面配置的文件路径，可以直接用定义的单个字母表示整个路径
      resolve: {
        extensions: ['.js', '.jsx', '.less', '.scss', '.css'],
        modules: [
          path.resolve(__dirname, 'node_modules'),
          path.join(__dirname, './src')
        ]
      },
    
      module: {
          rules: [{
              test: /\.(js|jsx)$/,
              use: ['babel-loader'],
              exclude: /node_modules/,
              include: path.join(__dirname, 'src')
          }, {
              test: /\.(less|css)$/,
              use: ["style-loader", "css-loader", "less-loader", "postcss-loader"]
          }, {
              test: /\.(png|jpg|gif|md)$/,
              use: ['file-loader?limit=10000&name=[md5:hash:base64:10].[ext]']
          }, {
              test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
              use: ['url-loader?limit=10000&mimetype=image/svg+xml']
          }],
      }
    };
    


  [1]: https://github.com/hyy1115/react-redux-webpack