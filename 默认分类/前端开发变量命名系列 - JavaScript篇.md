# [前端开发变量命名系列 - JavaScript篇](https://segmentfault.com/a/1190000020039039)

JavaScript作为前端开发从业人员必须掌握的3大基础知识中最重要的一环，也是平是接触时间最长、写得最多的。在开发过程中必然会遇到命名的问题，你会词穷、纠结、惆怅吗？本文的出现相信能够解决大部分烦恼，让你轻松写出符合规范、易读、简短的代码。

本文将通过大量的实例来试图自圆其说，形成一套系统化、实用的变量命名理化体系。通过按JavaScript的数据类型分类着手、细到一个函数的参数命名，并提供众多可选方案，并尽量给出其适用范围和利弊。

> 需要注意的是由于个人写作水平、和知识有限，很多方面叙述上有些生硬，在分类上也没有什么特别的依据，文章也没有人审稿，所以有什么纰漏还请留言告知。由于写作仓促，内容可能不全，后续会随着工作和学习的深入而不断地调整和更新。

# 布尔值(Boolean)命名

Boolean值是两种逻辑状态的变量，它包含两个值：**真**和**假**。在JavaScript中对应 `true` 和 `false`，在实践中通常使用数字`1`表示真值，`0`来表示假值。

虽然Boolean的状态只有两种但是在命名时可以进一步分类，这里给出几种场景：

## 场景一：表示可见性、进行中的状态

**解释**：**可见性**在通常指页面中的元素、组件是否显示（或者组件挂载到DOM上，但并不可见）。**进行中**主要是说明某种状态是处于持续进行中。

推荐命名方式为 `is + 动词（现在进行时）/形容词`，同时这种方式也可以直接不写 `is`，但是为了更好的作区分，建议还是加上。

```
{
  isShow: '是否显示',
  isVisible: '是否可见',
  isLoading: '是否处于加载中',
  isConnecting: '是否处于连接中',
  isValidating: '正在验证中',
  isRunning: '正在运行中',
  isListening: '正在监听中'
}
```

> **注意**： 在Java中使用这种方式是有一定副作用的，为什么请移步：[为什么阿里巴巴禁止开发人员使用 “isSuccess” 作为变量名？](https://zhuanlan.zhihu.com/p/54458588)

## 场景二：属性状态类

**解释**：通常用来描述实体（例如：HTML标签、组件、对象）的功能属性，而且定法比较固定。

```
{
  disabled: '是否禁用',
  editable: '是否可编辑',
  clearable: '是否可清除',
  readonly: '只读',
  expandable: '是否可展开',
  checked: '是否选中',
  enumberable: '是否可枚举',
  iterable: '是否可迭代',
  clickable: '是否可点击',
  draggable: '是否可拖拽'
}
```

## 场景三：配置类、选项类

**解释**：主要是指组件功能的开启与关闭，功能属性的配置。

这是一种比较常见的情景，目前命名方式也有很多种，但是归纳起来也不多。推荐使用 `withXx` 来表示组件在基本功能形态外的其它功能，比如组件的基础功能到高级功能的开启；使用 `enableXx` 来表示组件某些功能的开启；使用 `allowXx` 来表示功能属性的配置；使用 `noXx` 用于建议功能使用者这个不建议开启。

```
{
  withTab: '是否带选项卡',
  withoutTab: '不带选项卡',
  enableFilter: '开启过滤',
  allownCustomScale: '允许自定义缩放',
  shouldClear: '是否清除',
  canSelectItem: '是否能选中元素',
  noColon: '不显示label后面的冒号',
  checkJs: '检查Js',
  emitBOM: 'Emit a UTF-8 Byte Order Mark (BOM) in the beginning of output files.'
}
```

> **注意**：如果嫌分类太多，可以只使用其中一种方式，比如在Typescript中使用了 `allownXx` 和 `noXx`。

除了上面这些带有特定的前置介词、动词方式外还有一些在语义上带有疑问性质的组合通常也是作为Boolean命名的一种参考。

```
{
  virtualScroll: '是否启用虚拟滚动模式',
  unlinkPanels: '在范围选择器里取消两个日期面板之间的联动',
  validateEvent: '输入时是否触发表单的校验'
}
```

# 函数命名

函数在不同的上下文中的叫法也不一样，在对象中称为方法，在类中有构造函数、在异步处理时有回调函数，还有立即执行函数、箭头函数、柯里函数等。

函数命名的方式常常是和业务逻辑耦合在一起的，但是在命名规则上也有一些常见的模式可以遵循。

## 场景一：事件处理

事件处理函数是前端平时用到最多的，包括浏览器原生事件、异步事件和组件自定义事件。在写法上最常见的两种命名分别为 `onXx`、`onXxClick`和`handleXx`、`handleXxChange`。

这里如何在二者之间选择，可以从二方面来归类。一是，原生事件采用 `onXx`，而自定义事件使用 `handleXx`。二是，事件主动监听采用 `onXx`，被动处理使用 `handleXx`。

从实践及三大主流框架的文档关于事件部分的内容来看，推荐使用 `handleXx` 这种方式，而在表单提交的时候通常采用 `onSubmit` 函数。

其实，在实际项目中很少严格这样来命名事件处理函数，因为这种方式有一定的局限性，比如点击按钮打开一个对话框，使用 `handleOpenDlg` 和 `onOpenDlg` 都没有直接写 `openDlg` 方便，如果页面有多个不同功能的对话框采用这种方式会显得变量名过长，而handle和on就显得没有必要了，比如 `hanldeOpenCommentDlg` 就没有 `openCommentDlg` 直白。

下面列出了一些约定成俗的适用例子：

```
{
  onSubmit: '提交表单',
  handleSizeChange: '处理分页页数改变',
  handlePageChange: '处理分页每页大小改变',
  onKeydown: '按下键'
}
```

## 场景二：异步处理

这里主要是指在写数据层服务、状态管理中的Action命名，以及Ajax回调的命名规则。

```
{
  getUsers: '获取用户列表',
  fetchToken: '取得Token',
  deleteUser: '删除用户',
  removeTag: '移除标签',
  updateUsrInfo: '更新用户信息',
  addUsr: '添加用户',
  createAccount: '创建账户'
}
```

命名主要围绕数据的增删查找来划分，获取数据通常是 `getXx` 和 `fetchXx`，在作者看来两者在使用上的区分在于 `getXx` 的数据来源不一定直接取自异步的原始数据，可能是加工处理后的，而 `fetchXx` 是直接拿的原始数据。当然在实际项目中并没有区分，看个人喜好。

`deleteXx` 主要用于数据删除，而 `removeXx` 在语义上是移除数据，通常情况数据是还存在的，只是没有显示或数据假删除。`updateXx` 是指数据更新操作，`addXx` 是在已有列表数据中添加新的子项、而`createXx` 主要是创建新的实例，比如新建一个账户。

## 场景三： 跳转路由

在实际开发过种中，比如在使用react-router/vue-router时，在模板或者JSX中可以直接在标签中写上目标地址，但有些时候跳转的目标地址是经过判断或者组合后的，并且通过事件触发跳转的，这个时候就需要一个函数来处理，那么在函数命名的时候可以考虑使用

```
{
  toTplDetail: '跳转到模板详情页面',
  navigateToHome: '导航到首页',
  jumpHome: '跳转首页',
  goHome: '跳转首页',
  redirectToLogin: '重定向到登录页',
  switchTab: '切换Tab选项卡',
  backHome: '回到主页'
}
```

推荐使用 `toXx` 和 `goXx` 这两种方式，如果不是在当前页面打开/跳转页面，可以使用 `toBlankTplDetail` 或者 `goBlankHome` 这种方式来进一步语义化。如果前端页面是位于Webview中，也可以考虑采用 `toNativeShare` 这种方式来命名。

## 场景四：框架相关特定方法

这里主要是针对前端3大主流流行框架，有一些命名是带有特定标识符的，还有就是一些生命周期的命名方式。

```
{
  formatTimeFilter: '在AngularJs和Vue中，通常用于过滤器命名',
  storeCtrl: '用于AngularJs定义控制器方法',
  formatPipe: '用于Angular中，标识管道方法',
  $emit: 'Vue中的实例方法',
  $$formatters: 'AngularJs中的内置方法',
  beforeCreate: 'Vue的生命周期命名',
  componentWillMount: 'React生命周期命名',
  componentDidMount: 'React生命周期命名',
  afterContentInit: 'Anuglar生命周期命名',
  afterViewChecked: 'Angula生命周期命名',
  httpProvider: 'AngularJs服务',
  userFactory: '工厂函数',
  useCallback: 'React钩子函数'
}
```

## 场景五：数据的加工

这类场景在处理列表的时候经常会碰到，比如排序、过滤、添加额外的字段、根据ID和索引获取特定数据等。

### 情形一：根据特定属性获取属性

这里可以参考DOM方法的命名，比如：`getElememtById()`、`getElementsByTagName()`、`getElementsByClassName()` 和 `getElementsByName()`，然后提炼出一个比较实用的模板：`getXxByYy()`。其中 `Xx` 可以是：`element`, `item`, `option`, `data`, `selection`, `list`, `options` 以及一些特定上下文的名字，比如：`user(s)`, `menu(s)` 等。`Yy` 相对来说比较固定，经常用到的就是 `id`, `index`, `key`, `value`, `selected`, `actived` 等。

除了使用 `get`，还可以使用 `query` 和 `fetch`，但是需要注意和上面提到的异步数据处理作一个区分。

```
{
  getItemById: '根据ID获取数据元素',
  getItemsBySelected: '根据传入的已选列表ID来获取列表全部数据',
  queryUserByUid: '根据UID查询用户'
}
```

注意：在 `getItemsBySelected` 这种情形下直接写成 `getItemsSelected` 更好，但只适用于 `Yy` 为形容词的场景

### 情形二：格式化数据

这里的格式化操作包括排序、调整数据结构、过滤数据、添加属性。命名通常使用 `formatXx`, `convertXx`, `inverseXx`, `toggleXx`, `parseXx`, `flatXx`, 也可以结合数组的一些操作方法来命名，比如 `sliceXx`, `substrXx`, `spliceXx`, `sortXx`, `joinXx` 等来命名。

```
{
  formatDate: '格式化日期',
  convertCurrency: '转换货币单位',
  inverseList: '反转数据列表',
  toggleAllSelected: '切换所有已选择数据状态',
  parseXml: '解析XML数据',
  flatSelect: '展开选择数据',
  sortByDesc: '按降序排序'
}
```

# 数组命名

数组的命名推荐使用复数形式来命名，还有就是名词和具有列表意思的单词组合。常见的词汇有 `options`, `list`, `maps`, `nodes`, `entities`, `collection` 等。

```
{
  users: '用户列表',
  userList: '用户列表',
  tabOptions: '选项卡选项',
  stateMaps: '状态映射表',
  selectedNodes: '选中的节点',
  btnGroup: '按钮组',
  userEntities: '用户实体'
}
```

# 选项元素、下拉元素命名

主要针对的是在下拉选择框、选项卡元素、Radio、Checkbox等数据源每个选项数据的命名。常见的词汇有：`title`, `name`, `key`, `label`, `field`, `value`, `id`, `children`, `index`, `nodes` 等。

基中 `title/name/key/label/field` 作为选项显示名, `value/id` 用于唯一标识选项，`children/nodes` 用于包含子节点内容。如果选项元素的语义很明确的情况下也可以直接使用特定单词来代替提到的这些泛指的词汇，例如菜单列表就可以使用 `menu` 来作为显示名。

```
// 最常见组合
{
  title: '标题',
  value: 'ID值'
}

// 组合二
{
  label: '标签名',
  value: 'ID值'
}

// 组合三
{
  name: '元素名',
  id: 'ID值'
}

// 组合四
{
  field: '字段',
  value: '标识',
  index: '索引'
}
```

# 当前选项、激活项命名

适用列表的选中项、菜单选中项、步骤操作的当前进行步骤、页面路由当前路由等的命名。

```
{
  activeTab: '当前选中选项卡',
  currentPage: '当前页',
  selectedData: '当前选项中数据',
}
```

# 临时数据、比对数据命名

针对在代码中有时会使用一些临时的变量来存储数据、保存数据快照的场景下的命名。

```
{
  swapData: '临时交换数据',
  tempData: '临时数据',
  dataSnapshot: '数据快照'
}
```

# 数据循环语句中变量命名

在使用 `for` 循环时，多层嵌套请依次使用 `i/j/k`，超过3层可以使用 `x/y/z`，`m/n` 来命名。使用 `forEach`, `map`, `filter`等方法时，每一个元素命名可以根据不同语境下的特殊名字来命名，比如遍历 `menus`，那么每个元素可以命名为 `menu`，不然则使用上下文无关的词汇，比如：`item`, `option`, `data`, `key`, `object` 等。至于索引通常使用 `index`，如果多层可以使用 `index + 数字` 的形式，也可以直接使用 `i/j/k` 来作为索引，甚至还可以使用 `subIndex/grandIndex` 这种方式来命名。

对于在使用 `for` 循环时数组长度如果需要单独命名的话，可以使用 `xxlength/xxLens`，或者 `xxCount`。

在循环的过程中通常还会统计某个条件下数据匹配的次数、重复元素数量、记录中间结果等情况。这里推荐使用 `count` 表示次数，`skipped` 表示跳过的数量，`result` 表示结果。

```
// for 循环
for (let i = 0; i < 10; i++) {
  for (let j = 0; j < 10; j++) {
    for (let k = 0; k < 10; k++) {
      // do something
    }
  }
}

for (let i = 0, lens = this.options.length; i < lens; i++) {
  // do something
}

// forEach
users.forEach((item, index) => {
  // do something
})

menus.forEach((menu, index) => {
  if (menu.children) {
    menu.children.forEach((subMenu, subIndex) => {
      if (subMenu.children) {
        subMenu.children.forEach((grandMenu, grandIndex) => {
          // 一个不常用的示例
        })
      }
    })
  }
})
```

# 方法参数命名

方法的参数名称和数量在不同的方法中各不相同，但是还是有一些固定的模式可以参考，比如在Vue中监听属性变化的新值和旧值；`reduce` 方法的上一个值，当前值；回调函数的命名、剩余参数的命名等。

## 情形一：新值、旧值

常见于Vue中`watch` 对像中的属性监听回调函数，推荐使用

```
{
  oldVal: '旧值',
  newVal: '新值'
}
```

## 情形二：上一个值、下一个值和当前值

这种情形见于路由的钩子方法，`Object.assign` 数据拷贝的参数。

```
// 组合一
{
  from: '从...',
  to: '到...'
}

// 组合二
{
  prev: '上一个...',
  next: '下一个...',
  cur: '当前'
}

// 组合三
{
  source: '源',
  target: '目标'
}

// 组合四
{
  start: '开始',
  end: '结束'
}
```

## 情形三：异步数据返回

用于Promise的`then`方法参数，`await` 的返回的Promise等。可选择的词汇有：`res`, `data`, `json`, `entity`，推荐使用 `res`。

```
demoPromise.then(res => {
  // do something
})
```

## 情形四：其它情况

一些使用不多，但是在编程时约定成俗的命名方式。例如 `ch` 表示单个字符，`str` 表示字符串, `n` 代表次数, `reg` 表示正则, `expr` 表示表达式, `lens` 表示数组长度, `count` 表示数量, `p` 表示数据的精度, `q` 表示查询(query), `src` 表示数据源(source), `no` 表示数字(number), `rate` 表示比率, `status` 表示状态, `bool` 表示布尔值, `arr` 表示数组值, `obj` 表示对象值, `x` 和 `y` 表示坐标两轴, `func` 表示函数, `ua`表示User Agent, `size` 表示大小, `unit` 表示单位, `hoz` 表示水平方向, `vert` 表示垂直方向, `radix` 表示基数，根

```
// 传入单个字符
function upper(ch) {}

// 数量重复
function repeat(str, n)

// 正则
'abab'.replace(reg, 'bb')
```

# 事件命名

这里根据DOM、nodejs和一些框架和UI组件的事件进行归纳

## **DOM事件**

这里列举DOM中常见的事件命名

```
{
  load: '已完成加载',
  unload: '资源正在被卸载',
  beforeunload: '资源即将被卸载',
  error: '失败时',
  abort: '中止时',
  focus: '元素获得焦点',
  blur: '元素失去焦点',
  cut: '已经剪贴选中的文本内容并且复制到了剪贴板',
  copy: '已经把选中的文本内容复制到了剪贴板',
  paste: '从剪贴板复制的文本内容被粘贴',
  resize: '元素重置大小',
  scroll: '滚动事件',
  reset: '重置',
  submit: '表单提交',
  online: '在线',
  offline: '离线',
  open: '打开',
  close: '关闭',
  connect: '连接',
  start: '开始',
  end: '结束',
  print: '打印',
  afterprint: '打印机关闭时触发',
  click: '点击',
  dblclick: '双击',
  change: '变动',
  select: '文本被选中被选中',
  keydown/keypress/keyup: '按键事件',
  mousemove/mousedown/mouseup/mouseleave/mouseout: '鼠标事件',
  touch: '轻按',
  contextmenu: '右键点击 (右键菜单显示前)',
  wheel: '滚轮向任意方向滚动',
  pointer: '指针事件',
  drag/dragstart/dragend/dragenter/dragover/dragleave: '拖放事件',
  drop: '元素在有效释放目标区上释放',
  play: '播放',
  pause: '暂停',
  suspend: '挂起',
  complete: '完成',
  seek: '搜索',
  install: '安装',
  progress: '进行',
  broadcast: '广播',
  input: '输入',
  message: '消息',
  valid: '有效',
  zoom: '放大',
  rotate: '旋转',
  scale: '缩放',
  upgrade: '更新',
  ready: '准备好',
  active: '激活'
}
```

## **自定义事件**

在封装组件时提供的事件名除了参考DOM事件外，在命名上也可以参考Github Api 采用 `动词过去时 + Event` 的方式, Visual Studio Code Api的 `on +

```
{
  assignedEvent: '分配事件',
  closedEvent: '关闭事件',
  labeledEvent: '标签事件',
  lockedEvent: '锁事件',
  deployedEvent: '部署事件'
}
```

此外，很多命名方式可以根据场景使用 `元素 + click` 、`元素 + change` 、`select + 范围`等方式灵活运用

```
{
  selectAll: '选择所有',
  cellClick: '当某个单元格被点击时会触发该事件',
  sortChange: '当表格的排序条件发生变化的时候会触发该事件'
}
```

# 状态管理命名

如果在项目中用到了状态管理(redux/vuex/ngrx)，下面给出一些ActionType，Mutation, Action的命名参考。

```
// Redux 的 actionType
LOAD_SUCCESS
LOAD_FAIL
TOGGLE_SHOW_HISTORY
ON_PLAY
ON_LOAD_START
FETCH_SONGS_REQUEST
RECEIVE_PRODUCTS

// ngrx
const SET_CURRENT_USER = '[User] Set current';
const ADD_THREAD = '[Thread] Add';
const UPDATE_TRIP_SUCCESS = 'Update [Trip] Success';
```

# 其它命名

```
// 日期、时间
// --------------------------------------------------------
sentAt: '发送时间'
addAt: '添加时间'
updateAt: '更新时间'
startDate: '开始日期'
endDate: '结束日期'
startTime: '开时时间'
endTime: '结束时间'
```

# 参考

文章的写作过程中参考大量优秀的文章、优秀开源项目的源码、流行框架的最佳实践指南以及一些SDK的接口文档。

- Vue.js风格指南
- [Angular风格指南](https://www.angular.cn/guide/styleguide)
- [小程序Api](https://developers.weixin.qq.com/miniprogram/dev/api/)
- [Github GraphQL API V4](https://developer.github.com/v4/)
- [Facebook GraphQL API Reference](https://developers.facebook.com/docs/graph-api/reference)
- 【文章】[变量命名指南](https://zhuanlan.zhihu.com/p/24286730)
- [VS Code Api](https://code.visualstudio.com/api/references/vscode-api)
- [tsconfig json](http://json.schemastore.org/tsconfig)
- [Typescript Compiler Options](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
- [Airbnb JavaScript Style Guides](https://github.com/airbnb/javascript)
- [style-guides javascript](https://github.com/Khan/style-guides/blob/master/style/javascript.md)