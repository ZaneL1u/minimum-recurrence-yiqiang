<template>
  <div class="pagination">
    <!-- 上 -->
    <button :disabled="pageNo == 1" @click="getPageNo(pageNo - 1)">
      上一页
    </button>
    <button
      v-if="startNumAndEndNum.start > 1"
      @click="getPageNo(1)"
      :class="{ active: pageNo == 1 }"
    >
      1
    </button>
    <button
      v-if="startNumAndEndNum.start > 2"
      @click="getPageNo(pageNo - continues)"
    >
      ···
    </button>
    <!-- 中间部分 -->
    <button
      v-for="(page, index) in generateMiddlePage"
      :key="index"
      @click="getPageNo(page)"
      :class="{ active: pageNo == page }"
    >
      {{ page }}
    </button>

    <!-- 下 -->
    <button
      v-if="startNumAndEndNum.end < totalPage - 1"
      @click="getPageNo(pageNo + continues)"
    >
      ···
    </button>
    <button
      v-if="startNumAndEndNum.end < totalPage"
      @click="getPageNo(totalPage)"
      :class="{ active: pageNo == totalPage }"
    >
      {{ totalPage }}
    </button>

    <button :disabled="pageNo == totalPage" @click="getPageNo(pageNo + 1)">
      下一页
    </button>

    <input
      type="number"
      v-model="inputPage"
      min="1"
      :max="total"
      @keydown="handleKeyDown"
    />
    <button @click="getPageNo(+inputPage)">跳转</button>

    <button style="margin-left: 30px">共 {{ total }} 条</button>
  </div>
</template>

<script>
export default {
  name: "Pagination",
  data() {
    return {
      inputPage: 1,
    };
  },
  props: ["pageNo", "pageSize", "total", "continues"],
  computed: {
    //计算出总页数
    totalPage() {
      //向上取整
      return Math.ceil(this.total / this.pageSize);
    },
    //计算出页码的起始和结束数字
    startNumAndEndNum() {
      const { continues, pageNo, totalPage } = this;
      //先定义两个变量存储起始数字与结束数字
      let start = 0,
        end = 0;

      if (continues > totalPage) {
        start = 1;
        end = totalPage;
      } else {
        //起始数字
        start = pageNo - parseInt(continues / 2);
        //结束数字
        end = pageNo + parseInt(continues / 2);
        if (start < 1) {
          start = 1;
          end = continues;
        }

        if (end > totalPage) {
          end = totalPage;
          start = totalPage - continues + 1;
        }
      }
      return { start, end };
    },
    //过滤掉小于起始页的页码
    generateMiddlePage() {
      let arr = [];
      //避免页面中同时使用 v-for和v-if
      for (let i = 0; i <= this.startNumAndEndNum.end; i++) {
        arr.push(i);
      }
      let temp = arr.filter((item) => {
        return item >= this.startNumAndEndNum.start;
      });
      return temp;
    },
  },

  methods: {
    handleKeyDown(event) {
      const maxNumber = Math.ceil(this.total / 10);
      const enteredNumber = parseInt(event.target.value + event.key);
      if (!isNaN(enteredNumber) && enteredNumber > maxNumber) {
        event.preventDefault();
      }
    },
    getPageNo(val) {
      //自定义事件子页面向父页面传参，计算属性值
      this.inputPage = val;
      this.$emit("getPageNo", val);
    },
  },
};
</script>

<style lang="less" scoped>
.pagination {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  width: 100%;

  input {
    text-align: center;
  }

  button {
    margin: 0 5px;
    background-color: #f4f4f5;
    color: #606266;
    outline: none;
    border-radius: 2px;
    padding: 0 4px;
    vertical-align: top;
    display: inline-block;
    font-size: 13px;
    min-width: 35.5px;
    height: 28px;
    line-height: 28px;
    cursor: pointer;
    box-sizing: border-box;
    text-align: center;
    border: 0;

    &[disabled] {
      color: #c0c4cc;
      cursor: not-allowed;
    }

    &.active {
      cursor: not-allowed;
      background-color: #409eff;
      color: #fff;
    }
  }
}

.active {
  background: skyblue;
}
</style>
