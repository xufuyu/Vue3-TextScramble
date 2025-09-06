# TextScramble 文字扰乱组件使用文档
![demo](demo.gif)
`* 注：示例图片内的淡入效果，默认情况下未启用。 *`

TextScramble 是一个可复用的 Vue 3 组件，用于创建文字扰乱动画效果。它可以将一段文本转换为另一段文本，并在转换过程中显示扰乱动画效果。

## 安装和设置

要使用 TextScramble 组件，您需要手动将 `TextScramble.vue` 文件放置在项目的 `src/components/` 目录中。组件文件路径为：`src/components/TextScramble.vue`。

## 基本用法

### 1. 导入组件

```vue
<script setup>
import TextScramble from '@/components/TextScramble.vue'
</script>
```

### 2. 在模板中使用
#### 常规方法
```vue
<template>
  <TextScramble 
    :phrases="['原始文本', '目标文本']" 
    :start-delay="5000"
    :scramble-duration="4000"
    tag="h1"
    auto-start
  >
    <h1>原始文本</h1>
  </TextScramble>
</template>
```
#### 精简方法（适用于大部分需求）
```vue
<template>
  <TextScramble
      :phrases="['原始文本', '目标文本']"
      tag="h1"
  >
  </TextScramble>
</template>
```

## 属性 (Props)

| 属性名               | 类型       | 默认值   | 描述                               |
|-------------------|----------|-------|----------------------------------|
| phrases           | string[] | 必需    | 要在动画中切换的文本数组，至少包含两个元素          |
| start-delay       | number   | 5000  | 动画开始前的延迟时间（毫秒）                 |
| scramble-duration | number   | 4000  | 动画持续时间（毫秒）                     |
| auto-start        | boolean  | true  | 是否自动开始动画                       |
| stop-after-first  | boolean  | true  | 是否在第一次转换后停止动画                |
| tag               | string   | 'div' | 包装元素的标签类型                      |
| className         | string   | ''    | 自定义 CSS 类名                     |
| showInitialText   | boolean  | true  | 是否在动画开始前显示初始文本（phrases[0]或插槽内容） |

## 事件 (Events)

| 事件名            | 参数 | 描述      |
|----------------|----|---------|
| scramble-start | 无  | 动画开始时触发 |
| scramble-end   | 无  | 动画结束时触发 |

## 方法 (Methods)

通过模板引用（ref）可以调用以下方法：

| 方法名             | 参数 | 描述       |
|-----------------|----|----------|
| startScramble   | 无  | 手动启动扰乱效果 |
| performScramble | 无  | 直接执行扰乱效果 |

使用示例：
```vue
<template>
  <TextScramble 
    ref="textScrambleRef"
    :phrases="['Hello', 'World']"
  />
  <button @click="triggerScramble">执行动画</button>
</template>

<script setup>
import { ref } from 'vue'
import TextScramble from '@/components/TextScramble.vue'

const textScrambleRef = ref(null)

const triggerScramble = () => {
  textScrambleRef.value.performScramble()
}
</script>
```

## 使用示例

### 1. 基本用法（显示phrases[0]内容）

```vue
<template>
  <TextScramble 
    :phrases="['Evan\'s Personal Website', '埃文的个人网站']" 
    :start-delay="5000" 
    :scramble-duration="4000"
    tag="h1"
    auto-start
  />
</template>

<script setup>
import TextScramble from '@/components/TextScramble.vue'
</script>
```

### 2. 使用插槽内容

```vue
<template>
  <TextScramble 
    :phrases="['Evan\'s Personal Website', '埃文的个人网站']" 
    :start-delay="5000" 
    :scramble-duration="4000"
    tag="h1"
    auto-start
  >
    <h1>自定义初始内容</h1>
  </TextScramble>
</template>

<script setup>
import TextScramble from '@/components/TextScramble.vue'
</script>
```

### 3. 禁用初始文本显示

```vue
<template>
  <TextScramble 
    :phrases="['Evan\'s Personal Website', '埃文的个人网站']" 
    :start-delay="5000" 
    :scramble-duration="4000"
    :show-initial-text="false"
    tag="h1"
    auto-start
  />
</template>

<script setup>
import TextScramble from '@/components/TextScramble.vue'
</script>
```

### 4. 多文本循环播放

```vue
<template>
  <TextScramble 
    :phrases="phrases" 
    :start-delay="3000" 
    :scramble-duration="2000"
    :stop-after-first="false"
    tag="h2"
  >
    <h2>{{ phrases[0] }}</h2>
  </TextScramble>
</template>

<script setup>
import TextScramble from '@/components/TextScramble.vue'

const phrases = [
  'Hello World',
  '你好世界',
  'こんにちは世界',
  'Bonjour le monde'
]
</script>
```

### 5. 手动控制动画

```vue
<template>
  <TextScramble 
    ref="scrambleRef"
    :phrases="['Click me', 'Clicked!']" 
    :auto-start="false"
    tag="p"
  >
    <p>Click me</p>
  </TextScramble>
  <button @click="startAnimation">开始动画</button>
</template>

<script setup>
import { ref } from 'vue'
import TextScramble from '@/components/TextScramble.vue'

const scrambleRef = ref(null)

const startAnimation = () => {
  scrambleRef.value.startScramble()
}
</script>
```

## 注意事项

1. **文本数组要求**：`phrases` 属性必须是一个包含至少两个元素的数组。
2. **标签一致性**：使用 [tag](.\TextScramble.vue#L11-L11) 属性确保动画前后的元素标签类型一致。
3. **初始文本显示**：当 [showInitialText](.\TextScramble.vue#L13-L13) 为 `true`（默认值）时，组件会显示初始文本：
   - 如果提供了插槽内容，则显示插槽内容
   - 如果没有插槽内容，则显示 `phrases[0]` 的内容
4. **禁用初始文本**：当 [showInitialText](.\TextScramble.vue#L13-L13) 为 `false` 时，不显示任何初始文本
5. **自动停止**：默认情况下，动画在完成第一次转换后会自动停止，如需循环播放，请设置 `:stop-after-first="false"`。
6. **性能考虑**：对于较长的文本，动画可能会影响性能，请适当调整动画参数。

## 自定义样式

可以通过 CSS 类名自定义扰乱字符的样式：

```css
.dud {
  color: #777;
}
```

或者通过 [className](.\TextScramble.vue#L9-L9) 属性添加自定义类名：

```vue
<TextScramble 
  :phrases="['Hello', 'World']"
  class-name="my-scramble"
>
  <h1>Hello</h1>
</TextScramble>
```
