---
title: vue 父子组件通信
date: 2020-3-29 01:06:50
tags:
- vuejs
- 组件通信
categories:
- vuejs
preview: 100
---


## 父到子

通过在父组件的prop属性向子组件传递数据


```vue
// 父组件
<template>
	<student :student="liming"></student>
</template>
```

```vue
// 子组件
<template>
	<div>
    姓名: {{ student.name }}
  </div>
</template>

<script>
	export default {
    props: {
      student: {
        type: Object,
        default: ()=>({}),
      }
    }
  }
</script>
```

## 子到父

子组件通过`$emit('eventName', data)`产生事件, 在父组件中监听此事件

```vue
// 父组件
<template>
	<MyInput @update:data="handleUpdate"></MyInput>
</template>

<script>
	export default {
		methods: {
      handleUpdate(data) {
        // do something
      }
    }
  }
</script>
```

```vue
// 子组件
<template>
	// 当el-input的输入框的值发生改变, 会触发input事件
	<el-input @input="$emit('update:data', value)"></el-input>
</template>

<script>
</script>
```



## 双向绑定数据

### v-model原理

用v-model实现, v-model只是语法糖, 以下两种写法等同

```vue
<input v-model="message" />

// 转换后：
<input
  v-bind:value="message"
  v-on:input="message=$event.target.value">
```

- v-bind把message变量赋值给input的value, 这是`input`接收数据
- v-on监听input事件, 当检测到数据变化, 修改message的值, 这是input发送数据
- 两者在一起实现了当input的value改变, 则修改message变量

实现思想: 统一数据源

### 方案1 用model属性

这个方案和上面的一样, 只不过v-model默认绑定属性名为value, 默认事件名为input, 这个方法可以**自定义prop和event**

```vue
// 父组件
<template>
	<SearchBar v-models="text"></SearchBar>
	<!--等价于-->、
	<SearchBar :search="text" @update:search="this.text = $event.target.value"></SearchBar>
</template>
<script>
	export default {
		data: ()=> {
      text: ''
    },
    methods: {
      handleUpdate(val) {
        this.text = val;
      }
    }
  }
</script>
```

```vue
// 子组件
<template></template>
<script>
	export default {
    name: "SearchBar",
    model: {
      prop: 'search',
      event: 'update:search'
    },
    props: {
      search: {
        type: String,
        
      }
    }
  }
</script>
```



### 方案2: 用setter, getter

```vue
// 子组件
<template>
    <input type="text" v-model="model">
</template>

<script>
  export default {
    props: {
      value: {
        type: String,
        default: '',
      },
    },

    computed: {
      model: {
        get () {
          return this.value
        },
        set (newVal) {
          this.$emit('update:value', newVal)
        },
      },
    },
  }
</script>
```

以上例子中, 父组件传来值为value, 如果直接修改子组件中的value，并不会告知父组件数据已修改, 所以设置一个**中间变量**, 把它的getter设置为value, **setter则通知父组件修改数据**

### 方案3: 使用watch

此方法也是通过监听属性改变, 然后用emit产生事件 实现的

```vue
// 父组件
<template>
    <input :value="searchText" @update:value="handleUpdate">
</template>

<script>
  export default {
    data: ()=> {
			searchText: ""
    },
    methods: {
      handleUpdate(val) {
        this.searchText = val;
      }
    }
  }
</script>
```



```vue
// 子组件
<template>
    <input v-model="text">
</template>

<script>
  export default {
    data: ()=> {
			text: ""
    },
    props: {
      value: {
        type: String,
        default: ''
      }
    }
    watch: {
      text(newVal, oldVal) {
        this.text = newVal;  // 修改子组件的值
        this.$emit('update:value', newVal)	// 并且通知父组件
      }
    }
  }
</script>
```

### 方案4: 使用 .sync 修饰符

事实上，`.sync`修饰符是一个简写，它做了一件事情

```vue
// 父组件
<template>    
	<children :msg.sync="parentMsg"></children>    
	
	<!-- 等价于 -->    
	<children :msg="parentMsg" @update:msg="parentMsg = $event"</children>    <!-- 这里的$event就是子组件$emit传递的参数 --> 
	</template>
```

只是少写了个update:[prop]的监听事件而已, 当然子组件该怎么写还怎么写



### minxin

为了实现双向数据绑定，每次创建组件时都需要写相同的代码，所以可以写一个mixin文件，增加重用性。

```vue
// formMixin.js
const formMixin = {
  model: {
    prop: "data",
    event: "update"
  },
  props: {
    data: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    formData: {
      get() {
        return this.data;
      },
      set(val) {
        this.$emit("update", val);
      }
    }
  }
};

export default formMixin;

```





