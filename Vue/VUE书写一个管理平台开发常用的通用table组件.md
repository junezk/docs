# VUE书写一个管理平台开发常用的通用table组件

来现在这公司一年了，一年时间里经手做的项目有六七个，不过呢大部分都是一些管理平台的功能，而管理平台做的最多的就是各种表格的展示了，所以在开发过程中，为了提高开发效率，封装一些通用的功能组件是十分有必要的，在这里我就把我在开发过程中所封装的表格组件分享一下，当然肯定是有很多不足的，因为到目前为止我还是有一些想法没有实现的，也希望可以互相交流一下，就当抛砖引玉了，砸到谁也别怨我啊0.0

## 开发环境

我这边使用的是vue全家桶+ElementUI，毕竟管理平台，没那么高的要求，版本的话随意，毕竟只是说明一种设计方法，通用的

## 需求分析

管理平台的表格页面一般包含这几部分功能：1.操作按钮（添加，批量删除等）；2.表格数据筛选项（常用的有select过滤，时间过滤，搜索过滤等）；3.表格主体；4.分页。

### 1.操作按钮设计

操作按钮的添加我这边想到的有两种方法，第一种：直接使用vue提供的[插槽](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcomponents-slots.html)功能，直接在外部定义按钮、样式，以及按钮的操作事件，通过插槽插入组件内部；第二种：组件外部定义一个按钮的对象，通过父子组件通信传递到table组件内部，组件内部对按钮的对象进行解析、渲染，大概格式如下：

```
[
    {
        name: 'addBtn',
        text: '新增', // 按钮文案
        icon: 'el-icon-plus', // 按钮图标
        style: 'primary', // 按钮样式（这里取element的按钮样式）
        class: 'addBtn', // 自定义按钮class
        func: 'toAdd' // 按钮点击事件
    }, {
        name: 'multiDelBtn',
        text: '批量删除',
        icon: 'el-icon-delete',
        style: 'danger',
        class: 'multiDel',
        func: 'toMultiDel'
    }
]
```

### 2.表格数据筛选项

筛选项的设计没想到有什么好的，就内设几个常用的就上面说的那些，然后通过参数判断是否展示，其他如果有定制需求可以外部定义，然后通过插槽插入

### 3.表格主体

表格主要包括两部分：表头和表格体，分开分析

表格头我这边设计是在组件外部定义配置项传递组件内部，组件内进行解析，格式如下：

```
[
    {
        prop: 'name', // 表格数据对应的字段
        label: '用户姓名', // 表格头展示的信息
        sortable: false,  // 这一列是否支持排序，true|false|'custom',可以不传（为true是前端排序，不过前端排序没什么意义，一半排序的话还是传‘custom’进行服务端排序）
        minWidth: '100', // 这一列的最小宽度（用minWidth是因为在表格宽度不够的时候有个限制不会变形，在宽度过大的时候又能够按照各列的比例进行伸展，perfect！）
    }, {
        prop: 'address',
        label: '住址',
        minWidth: '170'
    }, {
        prop: 'age',
        label: '年龄',
        sortable: 'custom'，
        minWidth: '80'
    }
]
```

表格体没什么，就参看element-ui的就行

### 4.分页

分页也没什么好说的，内置在组件内，少于一页不显示，在element组件库选一个自己需要的功能的分页

## 组件开发

根据需求对组件进行了封装

#### 1.操作按钮设计

```
<div class="header-operate">
    <!-- 操作按钮插槽 -->
    <slot name="operateBtn"></slot>
    <el-button
      v-for="(btnItem, btnIndex) in propConfig.btnList"
      :key="btnIndex"
      :class="item.class"
      :type="item.style"
      :icon="item.icon"
      @click="toEmitFunc(item.func)"
    >{{item.text}}</el-button>
</div>
```

#### 2.筛选项设计

```
<div class="header-filter">
    <!-- 筛选项插槽 -->
    <slot name="filter"></slot>
    <el-select
      class="filter-iterm"
      v-if="propConfig.showFilter"
      v-model="filter"
      size="small"
      :placeholder="propConfig.filterPlaceholder"
      :clearable="true"
      filterable
      @change="handleSelect"
    >
      <el-option
        v-for="(item, index) in propConfig.filterOptions"
        :key="index"
        :label="item.label"
        :value="item.value"
      ></el-option>
    </el-select>
    <el-date-picker
      v-if="propConfig.showDatePick"
      v-model="dateRange"
      class="filter-iterm"
      align="right"
      size="small"
      format="timestamp"
      value-format="timestamp"
      :type="propConfig.datePickType"
      :clearable="true"
      :start-placeholder="propConfig.startPlaceholder"
      :end-placeholder="propConfig.endPlaceholder"
      @change="handleTimerange"
    ></el-date-picker>
    <el-input
      class="table-search filter-iterm"
      v-if="propConfig.showSearch"
      size="small"
      :placeholder="propConfig.searchPlaceholder"
      v-model="search"
      @keyup.13.native="toSearch"
    >
      <i slot="suffix" class="el-input__icon el-icon-search" @click="toSearch"></i>
    </el-input>
</div>
```

#### 3.table主体设计

```
<el-table
    :data="tableData.tbody"
    style="width: 100%"
    @selection-change="handleSelection"
    @sort-change="handleSort"
    >
    <el-table-column v-if="tableData.isMulti" type="selection" width="50"></el-table-column>
    <template v-for="(item, index) in tableData.thead">
      <el-table-column
        :key="index"
        :prop="item.prop"
        :label="item.label"
        :width="item.width"
        :min-width="item.minWidth"
        :sortable="item.sortable"
      >
        <template slot-scope="scope">
          <span class="default-cell" :title="scope.row[item.prop]">{{scope.row[item.prop]}}</span>
        </template>
      </el-table-column>
    </template>
</el-table>
```

#### 4.分页设计

```
<el-pagination
    class="table-pagination"
    v-if="tableData.pageInfo.total > tableData.pageInfo.size"
    layout="total, prev, pager, next, jumper"
    :current-page.sync="tableData.pageInfo.page"
    :page-size="tableData.pageInfo.size"
    :total="tableData.pageInfo.total"
    @current-change="pageChange"
></el-pagination>
```

#### 5.接受传参与methods

```
props: {
    tableConfig: {
      type: Object,
      default: () => {
        return {}
      }
    },
    tableData: {
      type: Object,
      default: () => {
        return {
          thead: [],
          tbody: [],
          isMulti: false, // 是否展示多选
          pageInfo: { page: 1, size: 10, total: 0 } // 默认一页十条数据
        }
      }
    }
},
  methods: {
    toEmitFunc (funName, params) {
      this.$emit(funName, params)
    },
    toSearch () {
      this.toEmitFunc('setFilter', { search: this.search, page: 1 })
    },
    pageChange (val) {
      this.toEmitFunc('setFilter', { page: val })
    },
    handleSelection (val) {
      let cluster = {
        id: [],
        status: [],
        grantee: [],
        rows: []
      }
      val.forEach(function (element) {
        cluster.id.push(element.id)
        cluster.status.push(element.status)
        cluster.rows.push(element)
        if (element.grantee) cluster.grantee.push(element.grantee)
      })
      this.toEmitFunc('selectionChange', cluster)
    },
    handleSort (value) {
      this.toEmitFunc('setFilter', {
        prop: value.prop,
        order: value.order
      })
    },
    handleTimerange () {
        if (this.dateRange) {
            this.eventBus('setFilter', {
              startTime: this.dateRange[0],
              endTime: this.dateRange[1]
            })
        } else {
            this.eventBus('setFilter', {
              startTime: '',
              endTime: ''
            })
        }
    },
    handleSelect () {
      this.toEmitFunc('setFilter', {
        filter: this.filter
      })
    }
  }
```

看到这，你肯定说这不就是element的table的使用吗？嗯...你说的很有道理，我竟无法反驳0.0，下面我就加入一些自己的想法设计吧

先写demo，在写的过程中才能一步步完善不足

```
<!--外部引用文件-->
<template>
  <div class="home">
    <TableComponent :loading="loading" :tableConfig="tableConfig" :tableData="tableData"/>
  </div>
</template>

<script>
import TableComponent from '../components/TableComponent'
export default {
  name: 'home',
  components: { TableComponent },
  data () {
    return {
      tableConfig: {
        btnList: [
          {
            name: 'addBtn',
            text: '新增',
            icon: 'el-icon-plus',
            style: 'primary',
            class: 'addBtn',
            func: 'toAdd'
          }, {
            name: 'multiDelBtn',
            text: '批量删除',
            icon: 'el-icon-delete',
            style: 'danger',
            class: 'multiDel',
            func: 'toMultiDel'
          }
        ],
        showFilter: true,
        filterOptions: [],
        showDatePick: true
      },
      tableData: {
        thead: [],
        tbody: [],
        isMulti: false,
        pageInfo: { page: 1, size: 10, total: 0 }
      },
      loading: false
    }
  },
  created () {
    this.toSetTdata()
  },
  methods: {
    toSetTdata () {
      let that = this
      that.loading = true
      that.tableData.thead = [
        {
          prop: 'name',
          label: '用户姓名',
          minWidth: '100'
        }, {
          prop: 'age',
          label: '年龄',
          sortable: 'custom',
          minWidth: '92'
        }, {
          prop: 'address',
          label: '家庭住址',
          minWidth: '130'
        }, {
          prop: 'status',
          label: '账号状态',
          minWidth: '100'
        }, {
          prop: 'email',
          label: '邮箱地址',
          minWidth: '134'
        }, {
          prop: 'createdTime',
          label: '添加时间',
          minWidth: '128'
        }
      ]
      setTimeout(() => {
        that.tableData.tbody = [
          { "id": "810000199002137628", "name": "邓磊", "age": 23, "address": "勐海县", "status": "offline", "email": "y.rbuuy@ndtccqms.lv", "createdTime": 1560218008 },
          { "id": "650000197210064188", "name": "蔡刚", "age": 26, "address": "四方台区", "status": "online", "email": "r.ifypqovuzl@rcvfhg.ir", "createdTime": 1500078008 },
          { "id": "450000199109254165", "name": "蔡明", "age": 22, "address": "其它区", "status": "online", "email": "z.rbq@uqadfyx.ee", "createdTime": 1260078008 },
          { "id": "440000198912192761", "name": "曹明", "age": 25, "address": "其它区", "status": "online", "email": "h.mkmqo@dcj.ee", "createdTime": 1260078008 },
          { "id": "310000198807038763", "name": "侯静", "age": 21, "address": "莱城区", "status": "offline", "email": "u.xlkda@ckoicbhk.br", "createdTime": 1560078008 },
          { "id": "310000198406163029", "name": "谭涛", "age": 29, "address": "闸北区", "status": "offline", "email": "r.wyr@hmqqlafes.no", "createdTime": 1500078008 },
          { "id": "220000199605161598", "name": "罗秀兰", "age": 27, "address": "其它区", "status": "online", "email": "d.eggqvlbol@crqodjvdys.nu", "createdTime": 1560078008 },
          { "id": "37000019810301051X", "name": "黎敏", "age": 27, "address": "其它区", "status": "online", "email": "s.unumfugq@qgeufl.om", "createdTime": 1560078008 },
          { "id": "440000201411267619", "name": "黄强", "age": 24, "address": "合浦县", "status": "offline", "email": "q.ufollx@kdvgtb.al", "createdTime": 1560218008 },
          { "id": "440000200504038626", "name": "叶艳", "age": 25, "address": "大渡口区", "status": "offline", "email": "h.trtiurcut@vnypp.sm", "createdTime": 1560078008 }
        ]
        that.tableData.isMulti = true
        that.tableData.pageInfo = { page: 1, size: 10, total: 135 }
        that.loading = false
      }, 500) // 模拟请求延时
    }
  }
}
</script>
```

demo写完就发现，很多功能都么得啊，功能很是单一，发现问题：

- 时间那一列，后端不一定传递过来就是可以直接展示的数据，如果传过来的是个时间戳呢？这时候就需要前端来做一下格式化处理
- 状态那一列，单纯的文字并不显眼，很多时候需要一个状态标签或者其他样式来展示
- 我还遇到很多次交互设计要求点击列表中姓名（不一定是姓名，就是某一项点击能进入详情页面）进入该用户的详情页
- 如果列表内有操作项呢？操作项该如何配置传递到组件内部？

有问题了，一个一个来0.0

第一个：时间那一项需要格式化，在表格中这一列都需要按照同样的方法进行格式化处理，那么我们可以把配置信息放在表头中，然后再表格组件中解析处理，定义formatFn：

```
{
  prop: 'createdTime',
  label: '添加时间',
  minWidth: '128',
  formatFn: 'timeFormat'
}
```

组件内添加formatFn判断

```
<template slot-scope="scope">
  <span
    class="default-cell"
    v-if="!item.formatFn || item.formatFn === ''"
    :title="scope.row[item.prop]"
  >{{scope.row[item.prop]}}</span>
  <span
    v-else
    :class="item.formatFn"
    :title="formatFunc(item.formatFn, scope.row[item.prop], scope.row)"
  >{{formatFunc(item.formatFn, scope.row[item.prop], scope.row)}}</span>
</template>
```

添加utils类，编写格式化数据方法，并注册全局

```
// 格式化方法文件（format.js）（记得要在main.js注册啊）
export default {
  install (Vue, options) {
    Vue.prototype.formatFunc = (fnName = 'default', data = '', row = {}) => {
      const fnMap = {
        default: data => data,
        /**
         * 时间戳转换时间
         * 接受两个参数需要格式化的数据
         * 为防止某些格式化的规则需要表格当前行其他数据信息扩展传参row（可不传）
         */
        timeFormat: (data, row) => {
          return unixToTime(data) // unixToTime是我书写的一个时间戳转时间的方法，如果你项目有引用其他类似方法插件，在这里返回格式化后的数据就可以
        }
      }
      return fnMap[fnName](data, row)
    }
  }
}
```

这样如果有其他格式化规则的也可以通过自定义格式化方法，然后再表头中定义需要调用的方法就可以了，同时这个格式化方法不只是可以用在表格中，其他任何你想要进行格式化的地方都可以在这个文件中定义，然后直接使用就可以了，而不用再引入方法再使用，是不是很方便

看到这里应该就明白这个table组件的核心其实还是这个格式化方法的使用

继续第二个问题，状态那一列，需要的不只是数据的变化，更是需要将相应的状态数据转换成对应的标签，这就需要扩展一下table组件，添加新的判断逻辑

```
{
    prop: 'status',
    label: '账号状态',
    minWidth: '100',
    formatFn: 'formatAccountStatus',
    formatType: 'dom'
}

<template slot-scope="scope">
  <!--不需要处理-->
  <span
    class="default-cell"
    v-if="!item.formatFn || item.formatFn === ''"
    :title="scope.row[item.prop]"
  >{{scope.row[item.prop]}}</span>
  <!--需要处理为标签-->
  <span
    v-else-if="item.formatType === 'dom'"
    :class="item.formatFn"
    v-html="formatFunc(item.formatFn, scope.row[item.prop], scope.row)"
  ></span>
  <!--单纯的数据处理-->
  <span
    v-else
    :class="item.formatFn"
    :title="formatFunc(item.formatFn, scope.row[item.prop], scope.row)"
  >{{formatFunc(item.formatFn, scope.row[item.prop], scope.row)}}</span>
</template>
```

我本意是想着format成element-ui的标签，如

![img](https://user-gold-cdn.xitu.io/2019/6/9/16b3b8752a1dcf7a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

但是在写format方法的时候发现直接返回el-tag标签，然后经过v-html解析为html标签但是后面发现并不能按照预期的那样解析成element的标签，思考一番没有发现什么好的方法，没办法只能自己返回原生标签，定义class然后自己书写样式修改为标签的样子



定义状态关系表，添加format方法

```
const accountStatusMaps = {
  status: {
    online: '在线',
    offline: '离线'
  },
  type: {
    online: 'success',
    offline: 'warning'
  }
}
// 用户账号状态转标签
formatAccountStatus: (data, row) => {
  return `<span class="status-tag ${accountStatusMaps.type[data]}">${accountStatusMaps.status[data]}</span>`
}
复制代码
```

这样一般的样式format是可以满足了，但是一些比较复杂的需求自己手写样式就比较不方便了，但是我自己一时也没想到好的解决办法，再加上这样比较难搞的需求比较少，所以就一直放着了（可能这就是我技术成长有限的原因吧0.0），各位看官如果有什么好的建议方法的话欢迎来提啊

继续第三项，这个需求在后台管理平台很容易遇到，点击名称进入详情，本来我还是想使用format，format出一个a标签，然后href是想要跳往的地址，但是功能虽然是实现了，但是使用a标签有个很大问题就是页面跳出感太强，只能放弃这个方法，后来没什么好的办法，就想着在表头上做文章，新定义一种formatType => 'link',可选参数linkUrl定义跳转链接，修改table组件的template

```
<span
    v-if="item.formatType === 'link'"
    :class="item.formatClass || 'to-detail-link'"
    :title="scope.row[item.prop]"
    @click="toLink(item.linkUrl, scope.row)"
>{{scope.row[item.prop]}}</span>

toLink (url, row) {
  if (url) {
    this.$router.push(`${url}/${row.id}`)
  } else {
    this.$router.push(this.$route.path + '/detail/' + row.id)
  }
}

.to-detail-link {
  color: #1c92ff;
  cursor: pointer;
  &:hover {
    color: #66b1ff;
  }
}
```

需求满足了，但是只是个妥协之计，该怎么在format返回的标签字符串上面绑定方法呢，如有想法，不胜感激，解决了这个问题能让这组件功能提高一大步，因为这个详情功能还算通用，可以在组件兼容，但是如果是其他点击方法呢？总不能一点点的都兼容，这样就失去了封装组件的意义，因为兼容是兼容不完的。

继续继续，操作项在后台管理平台是不可缺少的，主要的问题就在于怎么传递、抛出点击方法，我这边是这么实现的

先修改template

```
<span class="table-operation" v-if="item.prop === 'operation' && scope.row.hasOwnProperty('operation')">
    <span
      class="text-btn"
      v-for="(item, index) in scope.row.operation"
      :class="item.type"
      :key="index"
      @click="toEmitFunc(item.event, scope.row)"
    >{{item.text}}</span>
</span>
```

设置操作项数据信息

```
computed: {
    operateConfig () {
      return {
        optType: {
          toEdit: {
            event: 'toEdit', // 操作按钮调用的方法
            text: '编辑', // 操作按钮展示的文案
            type: 'primary' // 操作按钮展示的样式
          },
          toDel: {
            event: 'toDel',
            text: '删除',
            type: 'danger'
          }
        },
        optFunc: function (row) {
        // 在线状态用户不能删除
          if (row.status === 'offline') {
            return ['toEdit', 'toDel']
          } else {
            return ['toEdit']
          }
        }
      }
    }
}
```

把一些通用的属性、方法抽离出来放在tableMixins里面，减少每次调用的书写

```
// tableMixins.js
// 表格方法的mixins
export default {
  data() {
    return {
      // 表格数据，具体参考接口数据
      tableData: {
        thead: [],
        tbody: [],
        isMulti: false,
        pageInfo: { page: 1, size: 10, total: 0 }
      },
      // 表格是否处于loading状态
      loading: true,
      // 多选，已选中数据
      selection: [],
      // 查询条件，包括排序、搜索以及筛选
      searchCondition: {}
    }
  },
  mounted: function () { },
  methods: {
    // 多选事件, 返回选中的行及每行的当前状态
    selectionChange(value) {
      this.selection = value
    },
    接口请求到数据后将数据传入这个方法进行thead、tbody、pageInfo等信息的赋值
    afterListSet(res) {
      let formData = this.setOperation(res)
      if (formData.thead) {
        this.tableData.thead = JSON.parse(JSON.stringify(formData.thead))
      }
      this.tableData.tbody = formData.tbody
      if (formData.pageInfo) {
        this.tableData.pageInfo = JSON.parse(JSON.stringify(formData.pageInfo))
      }
      formData.isMulti && (this.tableData.isMulti = formData.isMulti)
      let query = JSON.parse(JSON.stringify(this.$route.query))
      this.$router.replace({
        query: Object.assign(query, { page: this.tableData.pageInfo.page })
      })
      this.loading = false
    },
    // 遍历原始数据，塞入前端定义好的操作项方法序列，设置操作项
    setOperation(res) {
      let that = this
      let tdata = JSON.parse(JSON.stringify(res))
      if (that.operateConfig && that.operateConfig.optFunc) {
        for (let i in tdata.tbody) {
          let temp = that.operateConfig.optFunc(tdata.tbody[i])
          let operation = []
          for (let j in temp) {
            operation.push(that.operateConfig.optType[temp[j]])
          }
          that.$set(tdata.tbody[i], 'operation', operation)
        }
      }
      return tdata
    }
  }
}
```

这样一套组合拳下来，管理平台常用的表格功能基本都实现了，贴一下成果图（样式没怎么写，这个就根据自己的项目风格进行调整吧）

![img](https://user-gold-cdn.xitu.io/2019/6/9/16b3c5c670bb9556?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我把这个组件的源码上传到了我的GitHub上了，同时写的有demo，有兴趣的朋友可以clone到本地琢磨琢磨，同时欢迎提出代码的不足或者bug，不胜感激0.0，贴上 [源码地址](https://github.com/LinXiNotDream/TableComponentDemo)

## 思考

组件写完了，文章也水完了，反思一些不足，一个是怎么渲染出UI组件的标签，另一个是怎么在format出来的标签上面绑定方法，诚然组件开发过程中有很多妥协，这些妥协就是自身水平或者视野没能达到导致的恶果，把组件抛出来就是为了集思广益，这个文章对你有帮助，能帮助你提高那我会很高兴，如果你帮忙解决了我遇到的困难，帮助到我的提高那我会更加高兴，我为的不就是这个嘛，所以欢迎大家帮忙想解决方案

当然组件开发过程中我还是做了一些很人性化的设计，一个是提供了多冲筛选条件同时作用的方法，就是维护一个searchCondition，每次筛选起作用就添加到searchCondition中，然后把searchCondition传递给数据查询方法，这个方法可以在demo中尝试，控制台会有输出，另一个小功能就不得不吐槽一下很多开源组件的一个反人类设计，富文本编辑器很多人都用过吧，会有一个menu的配置项，如果这个配置项你什么也不穿他就默认展示全部的菜单项，但是如果你不想要其中一个菜单项，你在menu的配置中把这一项置为false.......然后所有的菜单全没了，我看过一些源码是直接使用传递进去的menu覆盖默认的menu配置项，那么我只是想取消其中一项，我就得把全部的菜单项全部配置一遍且全部置为true，这...不坑爹呢嘛，所以我在向组件内部传参的时候添加了一个方法，而是使用解构赋值，维护一个computed，代码如下：

```
computed: {
    propConfig () {
      // defaultConfig是默认配置项，tableConfig是父组件传递进来的配置项
      return { ...this.defaultConfig, ...this.tableConfig }
    }
}
```

这样默认配置项只会在父组件某一项有修改的时候进行变更，其他不变

就这些吧，希望会对大家有所帮助

