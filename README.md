# iosInputFix

```bash
# 起 demo
$ npm run dev
```

将 main.js 文件注释掉，查看不加作用的 demo。页面滑到底，点击 input 框，唤起软键盘，页面上去，收掉软键盘，页面不下来

iosInputFix 文件：

```js
/**
 功能描述：iOS 中若表单元素(input/select) 靠近页面底部（页面已经滑到底），或是弹框表单，唤起软键盘后再收起可能导致 bug
 表现为：页面元素点不中，点击没反应或点击错乱，页面往上滚动后不回弹
 原因：软键盘弹出，为了不遮挡表单，页面会向上滑动一段距离，若页面已经滑动到底，再向上滑动时就会出现问题，键盘收起后视图回落，DOM 结构却未回落，最终表现为点不中目标元素
 解决：为解决这个问题，会给所有或是指定区域内的会唤起软键盘的表单元素添加监听事件，软键盘收起后重新设置 scrollTop
 * @param {Dom} target 非必填，监听指定区域内的 input 和 select 表单元素，失焦时消除弹框影响，若不填，则默认 document
 */
export default function (target = document) {
  const isIOS = /(ipad|ipod|iphone)/.test(navigator.userAgent.toLowerCase())
  if (!isIOS) return

  const nodeNames = ["INPUT", "SELECT"]
  let timer = null

  // 切换 input 时不处理，防止输入框未收起时页面回落
  target.addEventListener('focusin', e => {
    if (nodeNames.indexOf(e.target.nodeName) > -1) {
      clearTimeout(timer)
    }
  })

  target.addEventListener('focusout', e => {
    if (nodeNames.indexOf(e.target.nodeName) < 0) return

    const scrollTop = document.documentElement.scrollTop || document.body.scrollTop
    timer = setTimeout(() => {
      // 如果无效，将这一行注释去掉
      // document.body.scrollTop = document.documentElement.scrollTop = 0
      document.body.scrollTop = document.documentElement.scrollTop = scrollTop
    }, 0)
  })
}
```

如果不监听 focusin 事件，先点击 select，然后两个 input 框切换，就会有问题

