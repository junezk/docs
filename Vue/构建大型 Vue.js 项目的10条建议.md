# 构建大型 Vue.js 项目的10条建议

下面是我在开发大型 Vue 项目时的最佳实践。这些技巧将帮助你开发更高效、更易于维护和共享的代码。

今年做自由职业的时候，我有机会开发了一些大型 Vue 应用程序。我所说的这些项目，Vuex store 超过十个，包含大量的组件（有时候几百个）和视图页面。对我来说这是个很有益的经验，因为我发现了很多有意思的模式，可以让代码拥有更好的伸缩性。我还必须修正一些导致著名的意大利面条式代码困境的错误实践。

因此，今天我将与你分享10个最佳实践，如果你正在处理大型代码库，我建议你参考这些方法。

## 1. 使用 slot, 让组件更强大，也更容易理解

最近我写了篇关于 Vue.js slot 的文章，它强调了 slot 如何使组件更易于重用和维护，以及为什么应该使用它们。

🧐 但是这与 Vue.js 大型项目有什么关系呢？一张图片通常胜过千言万语，所以我要给你描绘一幅关于我第一次后悔没有使用它们的画面。

有一天，我要创建一个 popup。乍一看并没有什么复杂的东西，只是包含了一个标题，一个描述和一些按钮。所以我所做的就是把一切配置一股脑当做 props 传进去。最终我定义了三个 prop，用于自定义组件，当用户单击按钮时将发送一个事件。So easy !

但是，随着项目的发展，团队要求我们在其中展示更多其他的新内容：表单字段、不同的按钮(取决于它显示在哪个页面上)、卡片、页脚，以及列表。我以为，如果继续使用 prop 来迭代这个组件也没啥问题。但是我滴个神哪，我是大错特错！组件很快变得非常复杂，难以理解，因为它包含了无数的子组件，用了太多的 prop，并发送了一堆的事件。我开始体验到了可怕的情况，当你做出一点改动，其他页面的某个地方就会崩溃！我仿佛造了一个 Frankenstein 怪人，而不是一个可维护的组件!🤖

然而，如果我从一开始就依赖 slot，情况可能会好多了。最后我重构了所有东西，得到了这个小组件：更容易维护，理解起来更快，更好扩展！

```
<template>
  <div class="c-base-popup">
    <div v-if="$slot.header" class="c-base-popup__header">
      <slot name="header">
    </div>
    <div v-if="$slot.subheader" class="c-base-popup__subheader">
      <slot name="subheader">
    </div>
    <div class="c-base-popup__body">
      <h1>{{ title }}</h1>
      <p v-if="description">{{ description }}</p>
    </div>
    <div v-if="$slot.actions" class="c-base-popup__actions">
      <slot name="actions">
    </div>
    <div v-if="$slot.footer" class="c-base-popup__footer">
      <slot name="footer">
    </div>
  </div>
</template>

<script>
export default {
  props: {
    description: {
      type: String,
      default: null
    },
    title: {
      type: String,
      required: true
    }
  }
}
</script>
```

我的观点是，从经验来看，那些知道何时使用 slot 的开发人员所构建的项目确实会对其未来的可维护性产生很大的影响。由于发送的事件更少，代码更容易理解，而且提供了更大的灵活性，可以在其中显示任何组件。

> ️ 敲黑板：根据经验，当你开始在父组件中复制子组件的 prop 时，你就应该考虑使用 slot 了。

## 2. 合理组织 Vuex Store

通常，Vue.js 新手开始了解Vuex，因为他们刚好碰到了这两个问题：

- 组件树结构中相隔太远的组件之间访问数据
- 组件销毁后需要持久化数据

这个时候他们就会创建第一个 Vuex store，学习模块并开始在应用程序中组织它们。

问题在于，创建模块时没有单一的模式可以遵循。然而，我强烈建议你仔细考虑如何组织它们。据我所见，大多数开发人员更喜欢根据功能来组织它们。例如:

- Auth.
- Blog.
- Inbox.
- Settings.

就我来说，我发现根据从 API 获取的数据模型来组织它们更容易理解。例如:

- Users
- Teams
- Messages
- Widgets
- Articles

如何选择取决于你自己。唯一需要记住的是，从长远来看，一个组织良好的 Vuex store 会造就一个更高效的团队。它还将使新人在加入团队时更容易将他们的想法围绕在你的代码基础上。

## 3. 使用 action 发起 API 调用和提交数据

我的大部分 API 调用（如果不是全部）是在Vuex action 里面完成的。你可能会问：为什么要这么做？ 🤨 🤷‍♀️ 简单来说，它们中的大多数获取的数据需要提交到 store里去。另外，它们还提供了一层封装和可重用性，我很喜欢用。还有一些原因如下：

- 如果我需要在两个地方（假设是博客页面和首页）获取文章的第一页，我只需要用正确的参数调用合适的 dispatcher 就行了。除了 dispatcher 调用，无需重复代码就可以完成数据的获取，commit 到 store 和返回。
- 如果我需要写一些避免重复获取第一页的逻辑，我就可以在一个地方完成。这样做除了会减轻服务器负载，还会增强我对代码的信心。
- 我可以跟踪这些操作中的大多数 Mixpanel（一个网站用户行为分析工具） 事件，这使得分析代码非常容易维护。我确实有一些应用程序，其中所有的 Mixpanel 调用都是只在 action 中完成的。我不需要了解哪些数据被跟踪，哪些没有，以及什么时候发送这些信息。以这种工作方式的乐趣简直无法言说。

## 4. 用 mapState，mapGetters，mapMutations 和 mapActions 简化代码

通常不需要创建多个计算属性或方法，只需在组件内部访问 state/getters 或者调用 actions/mutations 。使用`mapState`，`mapGetters`，`mapMutations` 和 `mapActions` 可以帮助你简化代码，把来自 store 模块的数据分组到一起，让代码更容易理解。

```
// NPM
import { mapState, mapGetters, mapActions, mapMutations } from "vuex";

export default {
  computed: {
    // Accessing root properties
    ...mapState("my_module", ["property"]),
    // Accessing getters
    ...mapGetters("my_module", ["property"]),
    // Accessing non-root properties
    ...mapState("my_module", {
      property: state => state.object.nested.property
    })
  },

  methods: {
    // Accessing actions
    ...mapActions("my_module", ["myAction"]),
    // Accessing mutations
    ...mapMutations("my_module", ["myMutation"])
  }
};
```

有关以上工具方法的所有信息都在 [ Vuex 官方文档](https://vuex.vuejs.org/guide/modules.html)。🤩

## 5. 使用 API 工厂

我通常喜欢写一个`this.$api` 助手，以便在任何地方调用，获取后台 API 资源。在我的项目根目录有一个`api` 文件夹，包含了所有相关的类。如下所示（仅部分）：

```
api
├── auth.js
├── notifications.js
└── teams.js
```

每个文件都将其类别下的所有 API 资源分组。下面是我在 Nuxt 应用中使用插件初始化这个模式的方法（在标准的 Vue 应用中的过程也类似）。

```
// PROJECT: API
import Auth from "@/api/auth";
import Teams from "@/api/teams";
import Notifications from "@/api/notifications";

export default (context, inject) => {
  if (process.client) {
    const token = localStorage.getItem("token");
    // Set token when defined
    if (token) {
      context.$axios.setToken(token, "Bearer");
    }
  }
  // Initialize API repositories
  const repositories = {
    auth: Auth(context.$axios),
    teams: Teams(context.$axios),
    notifications: Notifications(context.$axios)
  };
  inject("api", repositories);
};


export default $axios => ({
  forgotPassword(email) {
    return $axios.$post("/auth/password/forgot", { email });
  },

  login(email, password) {
    return $axios.$post("/auth/login", { email, password });
  },

  logout() {
    return $axios.$get("/auth/logout");
  },

  register(payload) {
    return $axios.$post("/auth/register", payload);
  }
});
```

现在，我可以简单地在我的组件或 Vuex action 里像这样调用它们：

```
export default {
  methods: {
    onSubmit() {
      try {
        this.$api.auth.login(this.email, this.password);
      } catch (error) {
        console.error(error);
      }
    }
  }
};
```

## 6. 使用 $config 访问环境变量（在模板里特别有用）

你的项目可能在一些文件中定义了一些全局配置变量：

```
config
├── development.json
└── production.json
```

我喜欢通过 `this.$config` 助手快速访问它们，特别是在模板里。像往常一样，扩展 Vue 对象非常容易：

```
// NPM
import Vue from "vue";

// PROJECT: COMMONS
import development from "@/config/development.json";
import production from "@/config/production.json";

if (process.env.NODE_ENV === "production") {
  Vue.prototype.$config = Object.freeze(production);
} else {
  Vue.prototype.$config = Object.freeze(development);
}
```

## 7. 按照某个约定来给代码提交命名

随着项目的增长，你可能需要定期浏览组件的历史记录。如果你的团队没有遵循相同的约定来命名他们的提交，那么理解每个提交将会变得更加困难。

我一直推荐使用 [Angular 提交信息指南](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-guidelines)。我在每个项目中都遵循它，在很多情况下，其他团队成员很快就会发现遵循它带来的好处。

遵循这些指导原则可以得到更具可读性的信息，这使得在查看项目历史记录时更容易跟踪提交。简而言之，它是这样工作的：

```
git commit -am "<type>(<scope>): <subject>"

# 举例
git commit -am "docs(changelog): update changelog to beta.5"
git commit -am "fix(release): need to depend on latest rxjs and zone.js"
```

看下他们的 [README 文件](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-guidelines) 了解更多了解更多关于它和相关约定。

## 8. 项目上线后固定 package 版本

我知道，所有 package 都应该遵循[语义化版本规则](https://semver.org/)。但现实情况是，有些根本没有遵守。

为了避免因为某个依赖项破坏了整个项目而不得不在半夜醒来，锁定所有 package 版本可以让你的早晨工作压力更小。

它的意思很简单：避免使用带 `^` 前缀的版本号：

```
{
  "name": "my project",

  "version": "1.0.0",

  "private": true,

  "dependencies": {
    "axios": "0.19.0",
    "imagemin-mozjpeg": "8.0.0",
    "imagemin-pngquant": "8.0.0",
    "imagemin-svgo": "7.0.0",
    "nuxt": "2.8.1",
  },

  "devDependencies": {
    "autoprefixer": "9.6.1",
    "babel-eslint": "10.0.2",
    "eslint": "6.1.0",
    "eslint-friendly-formatter": "4.0.1",
    "eslint-loader": "2.2.1",
    "eslint-plugin-vue": "5.2.3"
  }
}
```

## 9. 在显示大量数据时使用 Vue Virtual Scroller

当你需要在某个页面中显示大量的行，或者需要循环大量的数据时，你可能已经注意到页面可能会很快变得非常慢。要解决这个问题，你可以使用[vue-virtual-scoller](https://github.com/akryum/vue-virtual-scroller)。

```
npm install vue-virtual-scroller
```

它将只渲染列表中可见的项，并重用组件和 dom 元素，效率高，性能好。它真的很容易使用，如丝般顺滑！✨

```
<template>
  <RecycleScroller
    class="scroller"
    :items="list"
    :item-size="32"
    key-field="id"
    v-slot="{ item }"
  >
    <div class="user">
      {{ item.name }}
    </div>
  </RecycleScroller>
</template>
```

## 10. 跟踪第三方包的大小

当很多人在同一个项目中工作时，如果没人关注安装包的数量，那么很快就会越来越多。为了避免应用程序变慢(特别是在移动网络变慢的情况下)，我在 VS Code 中使用了 [import cost 插件](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)。这样，我就可以从我的编辑器中看到导入的模块库有多大，并且可以在它变得太大时检查出问题。

例如，在最近的一个项目中，整个 lodash 库被导入(大约有24kB的gzipped)。结果只使用了 cloneDeep 方法。通过 import cost 插件定位到这个问题，我们是这样解决的：

```
npm remove lodash
npm install lodash.clonedeep
```

cloneDeep 函数可以在需要的地方引入：

```
import cloneDeep from "lodash.clonedeep";
```

> ️ 要进一步优化，你可以使用 [Webpack Bundle Analyzer ](https://www.npmjs.com/package/webpack-bundle-analyzer)，用交互式的可缩放地图可视化文件大小。

![Webpack Bundle Analyzer](构建大型 Vue.js 项目的10条建议.assets/16e643bc4e63d0b3)

关于处理大型 Vue 代码库，你还有其他最佳实践可以分享的吗？欢迎在评论区留言。