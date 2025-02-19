# 智慧商城

emm，看黑马vue视频跟着做的vue2项目，应该只能算是练手项目。

一个基于vue2，axiso和vant2开发的**移动商城**项目

链接：https://projects.sanye.blog/wisdomShop

# 小兔鲜

## 介绍

itheima的vue3项目

链接：https://projects.sanye.blog/rabbit

## 项目结构

* 布局（layout）页面：
  * home页面
  * 一级分类页面
  * 二级分类页面
  * 商品详情页面
  * 购物车页面
  * 订单页面
  * 支付页面
  * 会员页面
* 登录页面（login）

### **布局页面**

布局页面除了有一个`router-view`来防止其他二级页面组件，还要一些固定不变的组件，其中比较有意思的就是`fixedNav`，监听页面滚动事件，当滚动一定值后，让`fixedNav`显示出来。还有就是Nav组件的购物车图标，鼠标放到上面会显示购物车列表，这个效果实现核心如下：

```css
.layer {
   transform-origin: top;//变换的中心是top
   transform: scale(1, 0);//x轴不变，y轴缩放为0
   transition: all 0.4s 0.2s;//添加过渡，过渡事件是0.4s，延迟0.2秒开始动画
}
```

然后鼠标放到购物车图标后令`transform: none`

同时还对删除购物车商品做了动画效果，道理也是一样的。

```css
.del {
  transform-origin: right;
  transform: scale(0, 1);
}
```

因为这些组件只会在layout页面中用到，所以被放到`layout/components`目录下。

虽然说一个页面对应一个组件，但是为了降低项目的耦合度，把一个页面分解为多个组件才比较合理。

### **home页面**

home页面无非就是通过api获取数据然后渲染页面，逻辑较为简单。

### **商品详情页面**

商品详情页面（detail）页面，比较有意思的地方就是，使用组件内的`beforeRouteUpdate`路由守卫解决了路由跳转，但是组件被复用，导致的页面不刷新问题。还要一个难点就是放大镜，感觉就是在炫技；其中的小图片部分也很有意思：

```html
 <ul class="small-image">
    <li
         v-for="(item, index) in picList"
         :key="index"
         :class="{ active: index === activeIndex }"
         @mouseover="activeIndex = index"
    >
        <img :src="item" alt="" />
    </li>
</ul>
```

可以看出，监听`li`的`mouseover`事件，这是一个dom原生事件，监听对象也确实是个dom对象，就好比：

```html
<input @input='(e)=>{console.log(e.target.value)}'>
```

还后就是具体的放大镜部分，项目中使用的是`vueuse`的api，但是要是面试问这个问的肯定是自己如何实现。

```js
//捕获要检测对象
const target = ref(null)
//数据是响应式的
function mouseInElement(target) {
  onMounted(() => {
    target.value.addEventListener('mousemove', (e) => {
      const rect = target.value.getBoundingClientRect()
      console.log(e.clientX - rect.left, e.clientY - rect.top)
    })
    target.value.addEventListener('mouseleave', () => {
      console.log('出去了')
    })
  })
}
mouseInElement(target)
```

- **`e.clientX`**：表示鼠标指针相对于**浏览器视口左边界**的水平距离
- **`e.clientY`**：表示鼠标指针相对于**浏览器视口顶部**的垂直距离

调用一个dom元素的`getBoundingClientRect()`方法，会返回一个domrect对象。

总之就是通过实时获取鼠标在dom元素中的位置，来不断修改放大镜组件中的背景图片的位置。

还有一个组件就是**商品规格组件**，选择用户选择了特定的商品规格就会导出一个规格对象，但是这个组件实现的原理如何，没有去了解。

### **一级，二级分类页面**

一级分类页面就是通过ajax请求获取数据然后渲染页面，没什么好说的；二级分类页面的特点就是实现了**按条件筛选商品**，并且借助elementplus提供的全局指令`v-infinite-scroll`实现了商品列表的无限滚动：

```html
<ul
  v-infinite-scroll="load"
  :infinite-scroll-disabled="disabled"
  class="body"
>
  <li v-for="item in list" :key="item.id">
    <router-link :to="`/detail/${item.id}`">
      <img v-lazy-load="item.picture" alt="" />
      <p class="name ellipsis">{{ item.name }}</p>
      <p class="desc ellipsis">{{ item.desc }}</p>
      <p class="price">{{ item.price }}</p>
    </router-link>
  </li>
</ul>
```

如果面试要问肯定就是问这个指令的实现原理，就是监听ul的触底事件，然后修改查询参数（page++），发送请求获取新的商品数据，与原来旧的数据拼接。

### **登录页面**

登录页面使用elementplus提供的`el-form`组件简化了表单校验的过程，如果需要手动实现的话，考察的只是就是`正则表达式`了。

每个表单项可以对应多个校验规则。

```js
const rules = {
  account: [
    { required: true, message: '用户名不能为空', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    {
      pattern: /^\S{6,15}$/,//正则表达式
      message: '密码必须是6-15位的非空字符', //提示的消息
      trigger: 'blur' //触发表单校验的方式是光标消失
    }
  ],
  private: [
    {
      //自定义表单校验规则
      validator: (rule, value, callback) => {
        if (formModel.value.checked == false) {
          callback(new Error('请勾选协议！'))
        } else {
          callback()
        }
      },
      trigger: 'change' //触发validator方式是表单的值改变
    }
  ]
}
```

### **订单页面**

比较有意思的就是点击`切换`配送时间，支付方式等，原理都是一样的：

```html
<span
  v-for="(item, index) in payment"
  :key="index"
  :class="{ active: activeIndex2 === index }"
  @click="activeIndex2 = index"
>{{ item }}</span>
```

### **支付页面**

支付页面比较有意思的就是倒计时了吧，借助定时器，每秒让总时间`-1`，但是这个操作不能对格式化字符串操作，只能对具体的数值操作，所以还需要我们格式化时间，把总的秒数，格式化为`分:秒`的形式。要实现这一点，我们还借助了`dayjs库`来格式化时间，还有`计算属性`，每当总秒数改变，就返回最新的格式化时间。

```js
export const countDown = () => {
  const router = useRouter()
  const Time = ref(0)
  const formatTime = computed(() => dayjs.unix(Time.value).format('mm分ss秒'))
  const start = (time) => {
    Time.value = time
    let n = setInterval(() => {
      Time.value--
      if (Time.value == 0) {
        clearInterval(n) //关闭定时器
        ElMessage.error('订单超时')
        router.push('/cartList') //提示订单超时然后跳转到购物车
      }
    }, 1000)
  }
  return { formatTime, start }
}
```

### **购物车页面**

在这个项目中，还持久化存储了商品信息，这样刷新，关闭页面，购物车中的商品数据也不会丢失，我们在离线的时候也能操作购物车。

只需要在登录的时候，将本地购物车商品数据合并到登录账号下的远程购物车就行。

当我在这么介绍的时候面试管问我，将商品数据存储在购物车不会有安全问题吗？确实购物车商品数据存储在本地，可以通过js修改商品价格，但是最终订单的生成，是后端通过`选中的商品id`和`对应的商品数量`计算得到的，并没有太大的安全问题。

# 咸虾米壁纸

## 介绍

基于vue3+uni-app开发的移动端壁纸应用，点击缩略图能进入到预览页面查看大图。

但是由于部署到github pages上图片因为跨域加载不出来，就没有部署。

## 项目结构

* home页面
* 分类页面

### home页面

就是发请求获取数据，然后渲染页面；

页面结构部分就是使用了一些uni-app内置的组件或者扩展的组件，还使用了自己编写的组件，然后使用的语法还是vue的语法，没什么好说的。

比较新奇的就是使用`backdrop-filter:blur()`属性做了磨砂效果。然后还使用了`渐变色堆叠`。

### category分类页面

没啥好说的，就是发请求获取数据然后复用`theme-item`组件，然后使用`grid`来布局这个组件，顺便使用uni-app提供的`onPullDownRefresh`，`onReachBottom`钩子实现了下拉刷新和触底加载，面试官要问就是问你自己如何实现这个。

### user页面

功能包括`显示用户ip`，查看我的下载，我的收藏，联系客服等。

比较有意思的就是如何获取用户ip吧，虽然在这个项目中是后端服务器实现的。

### wallpaperList页面

我的收藏，我的下载，图片搜索等所有需要**展示缩略图**的页面(除了home页面)，复用的都是这个页面（其实就是wallpaperList组件）。

**根据跳转到这个页面传递的参数不同（通过onLoad接收）**，来展示不同的`navigationBar`，来发送不同的请求。其实就类似`动态路由参数`。

然而所有需要`展示缩略图的页面`，都需要通过修改缩略图链接得到`大图链接`，然后为了在preview页面展示（**页面间数据共享**），所以需要把`图片对象数组`存储在pinia仓库中，然后点击缩略图查看大图的时候（跳转到preview页面），就直接使用pinia中的数据。

因为所有需要展示缩略图的页面**不会同时存在**，所以我们只设置一个`picList`，只需要确保当前处在**某个展示缩略图的页面**的时候，仓库中的`picList`中存储的是该页面的`图片对象数组`。

```js
//放在home页面，如果home页面没销毁则直接使用页面组件内的recommendList.value
//如果页面组件被销毁，重新切换到home，由于这个操作会先于请求的数据到达被执行
//所以最后存储进仓库的还是通过请求返回的数据
onShow(() => {
    usePic.setPicList(recommendList.value)
})
```

而且这样有个好处就是，`preview页面`不需要纠结到底应该从那个页面的picList中取数据展示，因为这些页面的`图片对象数组`都存储在picList中。

### notice页面

主要用来展示公告详情的页面，大概也就是通过请求获取数据然后渲染页面，使用了`uni-dateformat`来格式化事件，使用`rich-text`组件来解析富文本。

### preview页面

是这个**项目功能最复杂**的页面，实现了滑动，预览图片，查看图片详细信息，收藏图片，下载图片的功能。

主要是基于大小占满整个页面的`swiper`轮播图组件实现的（不需要自动轮播，移除autoplay，interval属性）。

有多个展示缩略图的页面，点击缩略图会都跳转到这里，从仓库中的`picList`中取数据（大图链接）来展示。

**并且只渲染当前预览的图片和它左右的两张图片，减少了http请求的次数，图片的按需渲染的方式是通过`v-if`实现的。**

```html
<swiper-item v-for="(i,idx) in usePic.picList" :key="i._id">
     <image :src="i.bigPic" mode="aspectFill"
          v-if="index==idx||(index+1)%usePic.picList.length==idx||(idx+1)%usePic.picList.length==index">          </image>
</swiper-item>
```

同时添加了遮罩层mask（显示时间和功能组件），但是又为了让mask不影响轮播图滑动，让mask的高度为0。

图片详情信息被放在`uni-popup`组件中，给壁纸评分则是使用了`uni-popup`组件和`uni-icons`星标组件，评分好分后则发送请求到后端。

然后下载功能是整个项目实现起来最难的功能，因为为了考虑到用户的各种选择并提供良好的提示，调用了多次`toast`还有`loading`，除此之外还有各种**通过回调实现的异步api**，导致代码可读性极其的差（回调函数地狱问题），我尽量**使用了promise来优化代码的可读性。**

如果是H5端提示长按图片保存。

如果是非H5端，先获取图片的临时下载地址，再开始下载图片。

下载图片的过程，如果是非H5端，会先询问是否授予下载权限，如果授予成功并确认下载则图片下载完毕，同步到服务器。

如果授权失败或者取消下载，则下载失败。分析失败的原因，如果是取消下载，则提示“下载取消”，如果是授权失败，则提示用户给与下载权限，如果用户同意了，则打开权限设置面板。

**关键的api**

```txt
uni.getImageInfo({
      src: paperObj.value.bigPic,//可以是相对路径，临时文件路径，存储文件路径，网络图片路径
      success({
        path //图片的本地临时下载地址
      }) {//}
})
//H5不支持，微信小程序，app均支持，初次下载需要询问用户权限
uni.saveImageToPhotosAlbum({
	filePath:path,//图片的本地临时下载地，通过uni.getImageInfo得到
	success(){},
	fail(err){}//拒绝授权触发的回调函数，或者授权失败后无权限再下载的时候触发，或者取消下载也会触发
})
uni.openSetting({})//让用户设置权限，调起客户端小程序设置界面，返回用户设置的操作结果，配置属性只有那三个回调函数
```

### search页面

可以根据用户输入的`关键词`搜索图片，并保存搜索记录。

