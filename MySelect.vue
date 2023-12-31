<template>
  <!--本组件将选择、分页功能进行组合
    使用示例：
    <MySelect :initValue="initValue"
              :valueList="valueList"
              :placeholder="placeholder"
              :multiple="multiple"
              @selectorChanged="(e)=>value=e"
              @selectorCleared="(e)=>value=e"/>
  -->
  <!--重要修改点：
        1 单选多选逻辑的重写：
          原组件：单选时，value是单一字符串，多选时，value是字符串数组---》
          现在：  单选时和多选，value都是字符串数组
        2 添加分页选择功能：依赖另一个自定义升级的分页组件MyPagination
  -->
  <div style="width: 100%;height: 100%;">
    <!--加class="elPaginationSelect"是为了让下拉框的宽度和分页器的宽度一致---因此你必须在使用的时候给出具体宽度(200px)-->
    <el-select :disabled="disabled" ref="select" class="elPaginationSelect1"
               clearable
               filterable
               multiple
               :allow-create="allowCreate"

               v-model="value"
               :placeholder="placeholder"

               :filter-method="filterMethod"
               @change="selectorChange"
               @clear="selectorClear"
               @visible-change="visibleChange">
      <el-option v-for="item in options" :value="item.value"
                 :key="item.value" :label="item.label"/>
      <!--在Select组件中，请使用:small-size="true"，一方面是防止Select组件嵌套（分页组件内部也包含一个Select），二是Select组件内使用分页的时候一般要尽可能小-->
      <MyPagination :currentPageNum="currentPageNum" style="margin-bottom: 0;position: relative;bottom: 0"
                    :pageSize="pageSize"
                    :total="total"
                    :small="true"
                    @update-currentPageNum="(e)=>currentPageNum=e"
                    @update-pageSize="(e)=>pageSize=e"/>
    </el-select>
  </div>

</template>

<script>
import MyPagination from "@/components/MyPagination.vue";

export default {
  name: "MySelect",
  components: {MyPagination},
  props: {
    disabled: {default: false},
    initValue: {// 需要初始化的数组
      default() {
        return []
      }
    },
    valueList: {// 下拉列表展示的数据
      default() {
        return []
      }
    },
    placeholder: {
      default: "请选择"
    },
    allowCreate: {
      default: false
    },
    multiple: {
      default: false
    }
  },
  data() {
    return {
      // Select
      value: [],// 单选/多选时 字符串数组
      valueListFilter: [],// 当用户进行搜索过滤时 它存的是过滤后的下拉列表// 当用户没有搜索时 它存的是原始完整的下拉列表

      // Pagination
      pageSize: 5,
      currentPageNum: 1,
    };
  },
  computed: {
    // 计算分页所需的数据：
    options() {// 分页
      if (!this.valueListFilter && this.valueListFilter.length === 0) return []
      return this.valueListFilter.slice(this.pageSize * (this.currentPageNum - 1), this.pageSize * this.currentPageNum)
    },
    total() {
      if (!this.valueListFilter) return 0
      return this.valueListFilter.length
    },
  },
  mounted() {
    this.value = this.initValue;
    this.valueListFilter = this.valueList
  },
  methods: {
    // select
    // 自定义搜索筛选方法 输入值发生变化时调用，参数为当前输入值 可参考ui官网的【远程搜索】例子
    filterMethod(query) {
      // console.log("filterMethod:", query)
      if (query === null || query === '') {// 没有输入值时，返回原有的下拉列表(all)
        this.valueListFilter = this.valueList;
      } else {
        // console.log("filterMethod字符串存在非空:", query)
        // 通过对比 输入值query \ 下拉选项的label值 来进行筛选数组值
        this.valueListFilter = this.valueList.filter((item) => {
          return item.label.toLowerCase().includes(query.toLowerCase());// 包含匹配，而不是完全匹配
        });
      }
    },
    // 选中值时调用，参数为当前选中的值，字符串数组
    selectorChange(val) {
      // 重新自定义单选、多选：
      if (!this.multiple) {// 定义单选：只能选中一个，选中后，下拉框关闭
        if (val.length === 1) {// 第一次选择
          this.value = [val[0]]// 单选时 value是字符串数组，但是长度为1
        } else if (val.length === 2) {// 前面已经有选择了，再次选择
          this.value = [val[1]]
        }
        // 关闭下拉框
        this.$nextTick(() => {
          this.$refs.select.blur()
        })
      } else {// 多选
        this.value = val
      }
      this.$emit("selectChanged", this.value);// 通过$emit触发父组件的事件，将其他逻辑交由父组件处理
    },
    // 清空时调用
    selectorClear() {
      this.value = []// 清空
      this.$emit("selectCleared", this.value);// 通过$emit触发父组件的事件，将其他逻辑交由父组件处理
    },
    // 下拉框出现、关闭时调用，参数为 true 表示打开 false 表示关闭
    visibleChange(visible) {
      if (!visible) {// 下拉框关闭 实现类似于【确认提交】的功能(由父组件完成) submit

      }
    },

  },
  watch: {
    // 监听父组件传来的数据：(初次运行不会触发)
    initValue(newVal) {
      this.value = newVal
    },
    valueList(newVal) {
      this.valueListFilter = newVal
    },
  },

};
</script>

<style scoped>

.elPaginationSelect .el-input input {
  border: none !important;
  width: 100%;
}

</style>
