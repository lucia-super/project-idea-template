## 1. 模型设计

在 MVVM 下面的模型设计，如果没有一定的准则，才开始开发很容易随意定节点，然后随意挂载

在经历我们项目的一轮重构之后， 我们的模型设计原则：

1. 添加 namespaced，用以隔离各个业务模块的 mutations， actions。
2. 结构类似，以 namespaced 及节点名字区别此 data model 属于哪个业务。
3. data model 中如果存在其他业务模型，则用 module 的方式挂载
4. 在使用的命名空间之后，很多方法其实都是类似的，比如获取列表页面，删除，增加，唯一不同的是里面的 API 名字
   针对 mutations 写全重用的方法，在业务中合并入可重用的所有方法
   针对 Action, 可抽象出所有的相似的点在重用的公共方法中
5. 在 Action 中覆盖动态更新的变动，如果在页面子页面做了删改增加的动作，需要刷新，则直接在 vm 中更新，页面不需要跟随变化做任何处理
6. 在公共重用的文件中包含：
   a. data model 中分页的结构
   b. mutation 中的获取数据（包含条件查询），修改数据
   c. action 中修改数据，删除数据，刷新列表数据

**_注意: 涉及到的重用的的方法和 json 数据结构较多，避免对象被引用，在给新的 data model 赋值结构的时候，要深度 clone_**

对象节点作为一级节点：

sample: students && schools

schools 包含的结构：

```
state.schools = {
    list: {
        data: [],
        pagination: {
            size: 10,
            currentPage: 1,
            amount: 0,
            totalPages: 0
        },
        conditions: {},
        loading: false
    },
    current: {}
}
```

```
state.students = {
    list: {
        data: [],
        pagination: {
            size: 10,
            currentPage: 1,
            amount: 0,
            totalPages: 0
        },
        conditions: {},
        loading: false
    },
    current: {
      data: {},
      loading: false
    }
}
```

参照以上结构，就会发现对象的结构是一样的，

```
{
    list: {
        data: [],
        pagination: {
            size: 10,
            currentPage: 1,
            amount: 0,
            totalPages: 0
        },
        conditions: {}
    },
    current: {
      data: {},
      loading: false
    }
}
```

所以这部分就是可重用的数据结构模型，这个我们只要在 reusable 的文件中定义一次

```
export const DEFAULT_PAGINATION = {
  size: 10,
  currentPage: 1,
  amount: 0,
  totalPages: 0
}

export function generateBaseListState() {
  return {
    list: {
      data: [],
      pagination: DEFAULT_PAGINATION,
      conditions: {},
      loading: false
    },
    current: {
      data: {},
      loading: false
    }
  }
}
```

然后在业务声明的地方引入 reusable 文件，然后使用 generateBaseListState 生成节点对象

```
state.schools = generateBaseListState()
state.students = generateBaseListState()
```

**_ 灵活性 _**

1. 如果当前对象不需要 current
   `state.students = {...omit(generateBaseListState(), ['current'])`
2. 如果当前对象需要挂载其他对象, 引入其他对象，比如 students
   ```
   import students from './students'
   state.schools = {
      ...generateBaseListState(),
      current: {
          students: students
      }}
   ```

以上基本的 model 结构就有了，那么针对 model 的操作，无非就是增删改查

mutation：

```
/**
 * 保存列表数据、分页信息和查询条件
 * @param {Object} state Vuex state
 * @param {Object} payload vuex action payload
 * @param {Array<Object>} payload.data 列表数据
 * @param {Object} payload.pagination 分页信息
 * @param {Object} payload.conditions 查询条件
 */
export function SAVE_LISTDATA_PAGINATION_AND_CONDITIONS(state, { data, pagination, conditions }) {
  state.list.data = data || []
  state.list.pagination = pagination || DEFAULT_PAGINATION
  state.list.conditions = conditions || {}
}
```

actions：

```
/**
 * 刷新当前列表
 * @param {Object} Object Vuex state
 * @param {String} methodName 需要dispatch的名字
 * @param {String} markedKey reload 的标记名字
 */
export function reloadCurrentList({ state, dispatch, commit }, methodName, markedKey) {
  const { pagination: { currentPage, size }, conditions } = state.list
  dispatch(methodName, {
    payload: {
      conditions,
      pagination: {
        page: 1,
        size: size * currentPage
      },
      isReload: true
    }
  })
  commit(markedKey, false)
}

```

这些方法都是放在 reusable 文件中的，在我们的业务 model 中，我们只需要引入这些 common 的方法就好了。

## 2. 使用

模型设计已经准备好了， 在调用的地方，我们会发现，所有的 list 获取的方法名字都可以声明成一样的，只是命名空间不一样， 这时候参考我的 structure 结构。

比如 list 继承了 Base，是所有 list 的父类，这时候 list 可以完成自己所有的增删该查功能，只是命名空间是一个参数，由子类配置。

比如获取数据： fetchDataList， 在实际的数据结构上是

school: school.fetchDataList()
students: students.fetchDataList()

在 Base->List 中只要调用 fetchDataList 就好了，
然后子类中做：this.moduleType = 'trainingCentre'

父类模型匹配：

```
computed: {
    ...mapState({
      list(state) {
        const modules = this.moduleType.split('/')
        let currentListState = cloneDeep(state[modules[0]])
        if (modules && modules.length > 1) { // 当在某个模块的非一级节点上时，需要根据模块名称一层层找到当前的state节点
          forEach(modules, (item, index) => {
            if (index === 0) return
            currentListState = currentListState[item]
          })
        }
        return currentListState ? currentListState.list : {}
      },
      domainArea: state => state.global.domainArea
    }),
    pagination() { return this.list.pagination || DEFAULT_PAGINATION },
    loading() { return this.list.loading || false },
    listData() { return this.list.data || [] }
  }
```

父类 Action 匹配：

```
...mapActions({
      fetchListData: function(dispatch, page = 1) {
        if (this.moduleType) {
          dispatch(`${this.moduleType}/fetchListData`, { payload: {
            pagination: { page, size: this.pageSize },
            conditions: this.conditions
          }})
        }
      }
    })

```

由此上可见，父类可以做很多 common 的东西，子类要做的就很清闲啦
