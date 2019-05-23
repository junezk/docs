# Vue：基于Vuex的数据管理



基于Vue设计中大型应用时，随着应用大小以及业务流程的膨胀，数据管理也必然成为其中的重要一环。数据管理主要包括数据的存取、刷新、传递等方面，其实这也是笔者一直很疑惑的一个点，包括在做Android应用时，数据管理应该是开发过程中很重要的一部分，网上却很少关于这方面的讨论或是框架（Vuex更多是提供数据共享功能，具体数据管理思想&实现封装还需自行把控），至少我没有发现，如果你知道什么更好的方法欢迎在评论中交流。

#### 数据分类

根据数据的作用域，大致可以分为三类：

1.  全局配置数据：与用户无关，应用的全局配置，这类数据可以考虑放在localStorage或是sessionStorage中。
2.  用户数据：与登录用户绑定的数据，作用于用户登录后的所有流程，例如用户的基本信息、状态等。
3.  流程数据：一个业务流程中的共享数据，具体又可细分为登录前和登录后流程数据。

根据数据的分类，主要需要注意离开作用域时以及重复进入作用域时数据的变化，简单来说，切换用户或者退出登录时，2、3两种数据应该被清空或者重置。

#### 提供全局唯一的访问点

这也是Vuex框架的约定，state数据只能通过mutation来变更，我更推荐把这种约定作为数据访问的准则，不管是使用Vuex还是localStorage、sessionStorage，都应该封层一个全局数据访问入口，这样会极大地方便数据的统一管理和追踪。试想一下，如果从RESTful接口获取的数据需要做逻辑处理再保存时会怎么样，如果没有这样的全局唯一的访问入口，数据的获取和保存散落在各个页面中，势必增加很多额外的工作量。（当然Vuex本身支持这样的约定可以不用做额外的封装）

    //封装用户数据的入口
    export default{
      getUserInfo(){
          if(this.userInfo) return this.userInfo; //统一封装的入口可以很方便的提供一些通用的逻辑处理
         this.userInfo = sessionStorage.getItem("userInfo");
         this.userInfo && (this.userInfo = JSON.parse(this.userInfo));
         return this.userInfo;
      }
    }


​    

#### 数据的存取&刷新

对于业务流程来说，只关心数据的获取结果，成功或者失败，所以一个功能完善的数据模块应该封装好数据的拉取、缓存、刷新，伪代码逻辑如下：

    DataModule.getData().then(() => {
    //数据获取成功，继续业务流程
    }).catch(() => {
    //数据获取失败，失败逻辑
    })


当然，数据获取方式、刷新机制可能多种多样，但还是可以抽象出通用的特性，如果从接口设计角度来看，简单地可以抽象为fetch（数据拉取）、isNeedRefresh（是否需要刷新）两个接口，不同的数据Repository自行实现这两个接口，缓存和获取可以统一处理，然后由统一的DateCenter调用，伪代码如下（这边主要是提倡数据中心的设计思想，以Java的角度来看更易理解，前端童鞋可直接跳过）：

    //用户数据仓库
    UserInfoRepository{
      fetch(){
      //userInfo的拉取逻辑
      }
      isNeedRefresh(userInfo){
      //userInfo是否需要重新fetch的判断逻辑
      }
    }
    
    //全局唯一的数据访问入口
    DataCenter{
      UserInfoRepository userInfoRepo;
      getUserInfo(){
        //这边只是伪代码示例，不考虑同步异步、userInfo的缓存细节
        if(userInfoRepo.isNeedRefresh(this.userInfo)){
          this.userInfo = userInfoRepo.fetch();
        }
        return this.userInfo;
      }
    }
    
    //业务流程中调用就很简单，不需要考虑数据刷新、拉取等细节
    DataCenter.getUserInfo()


当然，上述只是对问题的抽象，对应到Vuex框架的实现可以简单地使用action来完成UserInfoRepository的逻辑，实现代码如下：

    //Vuex实现逻辑
    export default{
      state:{
        userInfo:null
      },
      mutations:{
        //全局唯一的数据修改点
        saveUserInfo(state,userInfo){
          state.userInfo = userInfo;
        }
      },
      actions:{
        getUserInfo(context,req){
          if(context.state.userInfo && 无需更新判断逻辑){
            return Promise.resolve(context.state.userInfo);
          }else{
            //对于数据的远程拉取需要自行封装统一的请求方法
            return Axios.post(req).then(resp => {
              context.state.commit("saveUserInfo",resp);
              return Promise.resolve(context.state.userInfo);
            })
          }
        }
    }
    
    //Vue控件中使用
    this.$store.dispatch("getUserInfo",req).then(resp => {
    //数据获取成功逻辑
    }).catch(() => {
    //错误提示
    })


至此，对于外部业务系统来说，只需要关心数据获取之后的处理即可，每次获取到的也肯定是最新的数据。

#### 如何确保数据非空

对于正常的流程来说，Vuex已经能够比较好的解决数据的存取问题，But，用户有意或者故意刷新了一下页面，你就会发现，诶，我数据呢，打开console一看，一堆的飘红！这也是Vuex的一个硬伤，页面刷新之后保存在内存中的数据全部都会丢失。对于这个问题的解决，笔者暂时未想到很完美的方案，这里提供两种方式做参考，如果你有更好的方案欢迎留言交流。

    方式一：保存到sessionStorage或是localStorage
      mutations:{
        //全局唯一的数据修改点
        saveUserInfo(state,userInfo){
          state.userInfo = userInfo;
          sessionStorage.setItem("userInfo",JSON.stringify(userInfo))
        }
      },
      actions:{
        getUserInfo(context,req){
          if(!context.state.userInfo){
            context.state.userInfo = sessionStorage.getItem("userInfo");
            context.state.userInfo && (context.state.userInfo = JSON.parse(context.state.userInfo));
          } 
          ...同上
          }
        }
    
    缺点：1、变化分散到sessionStorage/localStorage中，管理难度加大
    2、sessionStorage/localStorage中的数据可能存在安全隐患，不推荐敏感数据的保存（目前在各大网站也未发现sessionStorage/localStorage中会保存很多数据）
    
    方式二：结合Vue-Router&控件生命周期来确保数据的获取
    对于全局的用户数据可在全局路由钩子中获取来确保
    router.beforeEach((to, from, next) => {
      if (如果是必需userInfo的页面) {
        Store.dispatch("getUserInfo")
          .then(() => {
            next();
          })
          .catch(() => {
            //跳转至错误页面
            next({ name: "error" }); 
          });
        return;
      }
      next();
    });
    
    控件中利用beforeMount来实现
    beforeMount() {
      Store.dispatch("getUserInfo")
          .then(() => {
            //正常业务逻辑流程
          })
          .catch(() => {
            //错误逻辑处理，这部分代码可以统一放到mixins或者Vuex中
          });
    }
    缺点：1、每个页面需要考虑清楚各个数据的必要性
    2、需要设计数据获取失败的交互逻辑，跳转错误页面等（这部分设计好问题倒不大）


#### 业务流程中的数据传递

页面间的数据传递很简单，利用Vue-Router的params、query即可，问题是传递之后页面刷新的问题，页面刷新之后params数据会丢失，但是query数据由于会拼接在url中可以保留下来，所以对于采用何种方式传递数据以及页面刷新之后的处理，都需要根据业务流程具体分析：

    对于query数据，适合传递简单数据，适用于数据的id、主键等，如交易记录的id，这样就可以在页面的生命周期钩子中根据query数据查出详细数据
    beforeMount() {
      Axios.post({recordId:this.$route.query.id}).then().catch();
    }
    
    而对于复杂业务数据的传递则需要使用params，但是页面刷新后params数据会丢失，这时可根据params是否存在判断是否跳出流程
    beforeMount() {
      if(this.$route.params.data){
        //继续业务流程
      }else{
        this.$router.back();//or push到流程入口页面
      }
    }
    跳出业务流程的处理方式主要有一个问题：对于嵌套流程，跳转到流程入口可能体验上不是很好。


对于流程中间的数据，个人不是很推荐保存到Vuex，除非你能确保管理好数据的生命周期，考虑到页面回退，用户重复进入流程，甚至极端情况下用户可能会手动输入网址跳转某个页面的情况，流程数据的缓存是有一定风险。当然，这不仅仅需要前端确保数据的准确性，还需要后端确保数据的校验拦截等。

#### 数据的生命周期维护

对于Vuex中缓存的用户数据or流程数据，都需要在切换用户or退出流程时重置，提供重置数据的功能很简单，只需要在Vuex中增加clearState的mutation即可，难点是需要理清楚什么时候情况哪些数据需要重置，推荐将不同生命周期的数据封装成不同的module，然后提供不同的clear方法，在各个变更点调用即可。

        clearState(state) {
          //这边使用了lodash重置每个key，直接对state赋值是不起重要的
          lodash.mapKeys(state, (val, key) => {
            lodash.unset(state, key);
            return key;
          });
        }


#### 数据绑定问题

上面解决了数据的存取问题，当用到Vuex保存的数据做页面展示时可能会碰到数据还未获取成功的问题，这时console中一堆的undefine报错，甚至可能导致页面渲染失败，解决方法可采用computed的方式获取缓存数据中的字段：

    <div>{{name}}</div>
    
    computed:{
      name(){
        return this.$store.state.userInfo && this.$store.state.userInfo.person && this.$store.state.userInfo.person.name
      }
    }


#### 结语

文中提出了数据管理过程中的一些问题，也给出了个人在实践过程中的解决思路，希望对大家有所帮助。当然，如果你有更好的方法，也欢迎留言交流。