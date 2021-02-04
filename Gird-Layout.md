### Gird Layout 使用记录

iotechn-app 使用 vue-cli 脚手架创建，安装依赖的坑：没有node-sass sass-loader依赖

1. 使用npm安装用npm安装的依赖


	npm install --registry=https://registry.npm.taobao.org


2. 使用cnpm安装node-sass sass-loader依赖


	cnpm install node-sass sass-loader


3. 使用npm运行


	`npm run dev:h5`


​	
### Gird Layout 安装

	npm install --save vue-gird-layout --registry=https://registry.npm.taobao.org

### 参数

#### gird-layout

layout: 接受一个数组，数组长度与 item 长度一致，数组中每个对象需要拥有以下属性：

x: number, x坐标
y: number, y坐标
w: number,宽度非像素
h: number,高度非像素，指占多少格
i: string,id唯一字符串

auto-size: 容器是否适应内部变化 Boolean  非必填 默认为 true

col-num: layout被均分为横向多少格

row-height: 每一格有多高px

is-draggable: 是否可以移动的

is-resizeable: 是否能够改变容器大小

vertical-compact: 垂直方向上 是否应该紧凑布局 Boolean 非必填 默认为true

margin: 数组，相当于给格子定义内边距，应该遵循CSS的margin方位

use-css-transforms: 是否使用css的transforms来排版 非必填 为false时 使用后采用定位方式来布局 默认为true

#### Gird-Item
x,y,w,h,i 意义与layout中一致，直接绑定即可，相当于形成layout item js 三个绑定。

事件:
@resize 当item大小被改变时
@move 当item被移动时
@resized 当item已经被改变时
@moved 当item已经被移动时

方法签名：

```js
moveEvent: function(i, newX, newY,e){ 
},
resizeEvent: function(i, newH, newW){
},
movedEvent: function(i, newX, newY,e){
},
resizedEvent: function(i, newH, newW, newHPx, newWPx){
}
```
