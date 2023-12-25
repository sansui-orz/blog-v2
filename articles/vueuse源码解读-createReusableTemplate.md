# vueuse源码解读-createReusableTemplate

[tag]:VueUse|Vue3
[create]:2023-12-25

![Vue](//ju-stone-pro.oss-cn-guangzhou.aliyuncs.com/theme/1701071026358_8c7dd922ad47494fc02c388e12c00eac.jpeg)

## 前言

- 为什么要去看vueuse源码？

因为不想要闭门造车，在过往的经历中，我基本在小公司，身边没有什么大神，所以平时写代码都是凭自己的感觉。但是其实代码写多了就会发现会越写越乱，缺少前期的设计，以及缺乏对好代码的认知。所以我阅读源码的目标有两个：

1. 了解其中优秀设计理念
2. 了解其他优秀开发者如何写Vue

对于那些我觉得很棒的，我则会写一篇文章记录下来。

在用Vue的时候，我经常因为它的SFC模式而头疼，当组建中只有一小部分是需要复用的，也需要单独抽离成一个文件去复用。

这会产生很多琐碎的小文件，在后续编辑时寻找起来麻烦，维护时也不好维护。要是可以跟小程序一样，原地定义模版，或者跟jsx一样，可以通过文件内抽离出组件就好了（当然，Vue也可以使用jsx，但这不是主流所以不考虑这么做，只有手写js嵌套渲染函数，也不是一个好方法）。

而在看VueUse源码时发现，居然有类似于小程序一样的方式，原地定义模版进行复用，顿时感到眼前一亮。不错啊，才刚开始看，就了解到了之前自己没见过的用法。

## 用法

```html
<script setup>
import { createReusableTemplate } from '@vueuse/core'

const [DefineTemplate, ReuseTemplate] = createReusableTemplate()
</script>

<template>
  <DefineTemplate v-slot="{ $slots, otherProp }">
    <div some-layout>
      <!-- To render the slot -->
      <component :is="$slots.default" />
    </div>
  </DefineTemplate>

  <ReuseTemplate>
    <div>Some content</div>
  </ReuseTemplate>
  <ReuseTemplate>
    <div>Another content</div>
  </ReuseTemplate>
</template>
```

## 源码

```typescript
export function createReusableTemplate<
  Bindings extends object,
  Slots extends Record<string, Slot | undefined> = Record<string, Slot | undefined>,
>(
  options: CreateReusableTemplateOptions = {},
): ReusableTemplatePair<Bindings, Slots> {
  // compatibility: Vue 2.7 or above
  if (!isVue3 && !version.startsWith('2.7.')) {
    if (process.env.NODE_ENV !== 'production')
      throw new Error('[VueUse] createReusableTemplate only works in Vue 2.7 or above.')
    // @ts-expect-error incompatible
    return
  }

  const {
    inheritAttrs = true,
  } = options

  const render = shallowRef<Slot | undefined>()

  const define = defineComponent({
    setup(_, { slots }) {
      return () => {
        render.value = slots.default
      }
    },
  }) as unknown as DefineTemplateComponent<Bindings, Slots>

  const reuse = defineComponent({
    inheritAttrs,
    setup(_, { attrs, slots }) {
      return () => {
        if (!render.value && process.env.NODE_ENV !== 'production')
          throw new Error('[VueUse] Failed to find the definition of reusable template')
        const vnode = render.value?.({ ...keysToCamelKebabCase(attrs), $slots: slots })
        return (inheritAttrs && vnode?.length === 1) ? vnode[0] : vnode
      }
    },
  }) as unknown as ReuseTemplateComponent<Bindings, Slots>

  return makeDestructurable(
    { define, reuse },
    [define, reuse],
  ) as any
}
```

以上ts的代码看着比较复杂，稍微简化一下:

```js
export function createReusableTemplate(
  options = {},
) {
  const {
    inheritAttrs = true,
  } = options

  const render = shallowRef()

  const define = defineComponent({
    setup(_, { slots }) {
      return () => {
        render.value = slots.default
      }
    },
  })

  const reuse = defineComponent({
    inheritAttrs,
    setup(_, { attrs, slots }) {
      return () => {
        const vnode = render.value?.({ ...attrs, $slots: slots })
        return (inheritAttrs && vnode?.length === 1) ? vnode[0] : vnode
      }
    },
  })

  return [ define, reuse ]
}
```

简单去除多余代码之后，逻辑变得非常清晰。hook中创建了两个组件，其中一个用于接收组件定义，一个用以实际渲染。两者都是用defineComponent进行定义，通过修改渲染函数的方式，将实际渲染函数保存到作用域的变量中。

每天学到一点点，Good！