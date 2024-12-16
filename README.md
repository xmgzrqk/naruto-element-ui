基于 Element UI 进行了二次开发，在此非常感谢饿了么前端团队对项目进行了开源。

原项目地址：https://github.com/ElemeFE/element

目前主要对 Element 的 datePicker 组件进行了二次开发，修改如下：

## Feats
DatePicker
- 新增 type
  - quarter - 季度选择器
  - quarterrange - 季度范围选择器
  - weekrange - 周度范围选择器
- 新增季度日期格式
  - q 不补0 （范围：  1 - 4）
  - qq 补0 （范围：01 - 04）

## Install
```shell
npm install element-ui -S
```

## Quick Start
``` javascript
import Vue from 'vue'
import Element from 'element-ui'

Vue.use(Element)

// or
import {
  Select,
  Button
  // ...
} from 'element-ui'

Vue.component(Select.name, Select)
Vue.component(Button.name, Button)
```
<!-- For more information, please refer to [Quick Start](http://element.eleme.io/#/en-US/component/quickstart) in our documentation. -->

欢迎有这方面需求的开发者进行使用，如在使用过程中遇到任何问题或者发现 bug , 可通过下面方式联系我

邮箱：2848932856@qq.com

## LICENSE
[MIT](LICENSE)
