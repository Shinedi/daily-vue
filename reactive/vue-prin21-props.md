### Props (v2.6.11)
---
`Props` 作为组件的核心特性之一，也是我们平时开发 Vue 项目中接触最多的特性之一，它可以让组件的功能变得丰富，也是父子组件通讯的一个渠道。那么它的实现原理是怎样的，我们来一探究竟。

#### 规范化
---
在初始化 `props` 之前，首先会对 `props` 做一次 `normalize`，它发生在 `mergeOptions` 的时候，在 `src/core/util/options.js` 中：
```
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  // ...
  normalizeProps(child, vm)
  // ...
}

function normalizeProps (options: Object, vm: ?Component) {
  const props = options.props
  if (!props) return
  const res = {}
  let i, val, name
  if (Array.isArray(props)) {
    i = props.length
    while (i--) {
      val = props[i]
      if (typeof val === 'string') {
        name = camelize(val)
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    for (const key in props) {
      val = props[key]
      name = camelize(key)
      res[name] = isPlainObject(val)
        ? val
        : { type: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "props": expected an Array or an Object, ` +
      `but got ${toRawType(props)}.`,
      vm
    )
  }
  options.props = res
}

```
合并配置我们在组件化章节讲过，它主要就是处理我们定义组件的对象 `option`，然后挂载到组件的实例 `this.$options` 中。

我们接下来重点看 `normalizeProps` 的实现，其实这个函数的主要目的就是把我们编写的 `props` 转成对象格式，因为实际上 `props` 除了对象格式，还允许写成数组格式。

当 `props` 是一个数组，每一个数组元素 `prop` 只能是一个 `string`，表示 `prop` 的 `key`，转成驼峰格式，`prop` 的类型为空。

当 `props` 是一个对象，对于 `props` 中每个 `prop` 的 `key`，我们会转驼峰格式，而它的 `value`，如果不是一个对象，我们就把它规范成一个对象。

如果 `props` 既不是数组也不是对象，就抛出一个警告。
举个例子：
```
export default {
  props: ['name', 'nick-name']
}
```
经过 `normalizeProps` 后，会被规范成：
```
options.props = {
  name: { type: null },
  nickName: { type: null }
}
```
```
export default {
  props: {
    age: Number,
    sex: {
      type: String,
      default: 'female',
      validator: function (value) {
        return value === 'male' || value === 'female'
      }
    }
  }
}
```
经过 `normalizeProps` 后，会被规范成：
```
options.props = {
  age: { type: Number },
  sex: { type: String }
}
```
由于对象形式的 `props` 可以指定每个 `prop` 的类型和定义其它的一些属性，推荐用对象形式定义 `props`。

