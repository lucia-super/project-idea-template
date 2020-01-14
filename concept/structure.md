## 1. 工程化

**将前端项目当成一项系统工程进行分析、组织和构建，达到项目结构清晰、分工明确、团队配合默契、开发效率提高的目的**

在 ES6/VUE/REACT 中有很好的功能辅助我们结构化。

### 继承

如管理系统中，按照功能页面分类：

1. 列表页面： 搜索、排序、删除， 跳转到其他页面
2. 详情页面： 获取数据展示，跳转到其他页面
3. 编辑页面：获取数据，校验，保存，跳转到其他页面
   继承能很好的抽象出这几类页面的共同特性，在单个页面里面，我们只要关注特别的功能就行

![structure](https://github.com/lucia-super/project-idea-template/blob/master/concept/BASE.jpg "工程结构指导图")

### 混入

对于页面通用的样式，但是又不是强通用的，比如 formatter，validation 之类的 common 方法，可以通过混入的方式引入通用的文件。
formatter
validation
checkpermission

**注意这里一定要区别，那些方法在子类父类中都执行或者只在重写的方法里面执行**

对于 css, 用 less 或者用 sass 都要注意规划当前文件属于那个类别里的，
如果所有 list 中样式相似，那么对于 list 里基类来说就会有对应的 css

## 2. 组件化

组件具有独立性
页面上每个独立的、可视/可交互区域视为一个组件
每个组件可以对应一个工程目录，然后组件相关的资源在这个目录下就近维护
页面是组件的容器

**\*在项目中可以组件化的例子： 头部，导航，焦点图，侧边栏，底部，定制按钮，定制 form**

## 3. 模块化

功能通用的页面视为一个模块，这个功能通用的页面可以引用组件
js 的模块化方案 AMD/commonJS/UMD/ES6 Module
css 的模块化开发 Less/Sass/Stylus

## 4. 自动化

自动化是很大的方向，可以分为：

1.  自动化测试
    目前前端自动化单元测试工具: mocha/jest
    ui 测试（快照测试）: JEST // Enzyme
    项目集成测试： Selenium
2.  自动化构建
    a、基于任务打包工具，例如 Grunt 、Glup 。
    b、基于模块化打包工具，例如 browserify、Webpack、rollup.js 。
    c、整合型工具，例如 Yeoman、Fis、jdf 、athena、cooking 、weflow。
3.  自动化部署(CI)
    jenkins/docker

## 5. 跨平台/跨端

1.  跨平台
    a. 跨 android 和 ios: react native, weex, flutter
    b. 目前流行程度较高的是 react native，响应较快的是 flutter，flutter 也支持 web 端
2.  跨端
    小程序兴起了，各个平台都有自己的标准，比如微信，支付宝，字节跳动，百度等，如果一个应用依赖这几个平台，用跨端框架可以减少工作量
    unit-app>taro>chemeleon>wepy>mpvue
