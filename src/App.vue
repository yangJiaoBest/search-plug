<template>
  <div id="app">
    <div class="app-select">
      <h1>Products</h1>
      <a-input
        @change="searchChange"
        v-model="searchValue"
        placeholder="Search products"
      ></a-input>
      <br />
      <a-checkbox @change="onChange" v-model="checkValue">
        In stock products only
      </a-checkbox>
      <!-- 列表 -->
      <div class="list-content">
        <ul v-if="noData">
          <li class="noData">No products found.</li>
        </ul>
        <ul>
          <li v-for="item in dataList" :key="item.id">
            <h2 v-if="item.children.length">{{ item.category }}</h2>
            <div v-for="item2 in item.children" :key="item2.id">
              <p>
                <span :class="item2.inStock ? '' : 'noStock'">{{
                  item2.name
                }}</span>
                <span>{{ item2.price }}</span>
                <a-button @click="openModal(item2.imageUrl)"
                  >View detail</a-button
                >
              </p>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <!-- 图片弹窗 -->
    <a-modal
      v-model="visible"
      title=""
      @ok="handleOk"
    >
      <div class="img-content">
        <img :src="imageUrl" alt="" />
      </div>
    </a-modal>
  </div>
</template>

<script>
import debounce from "lodash/debounce";
import fetchProducts from "@/utils/MockProductApiClient";
export default {
  name: "App",
  data() {
    this.searchCommon = debounce(this.searchCommon, 400);
    return {
      dataList: [],
      beforeFilterData: [],
      searchValue: "",
      checkValue: false,
      noData: false,
      visible: false,
      imageUrl: null,
    };
  },
  async created() {
    const listData = await fetchProducts();
    // 处理默认数据列表
    let afterData = [];
    listData.forEach((item) => {
      let flag = afterData.find((item1) => item1.category === item.category);
      if (!flag) {
        afterData.push({
          category: item.category,
          children: [item],
        });
      } else {
        flag.children.push(item);
      }
    });
    this.dataList = afterData;
    this.beforeFilterData = afterData;
  },
  methods: {
    // 切换 In stock products only
    onChange(e) {
      this.checkValue = e.target.checked;
      this.searchCommon();
    },
    // 搜索
    searchChange(e) {
      this.searchValue = e.target.value;
      this.searchCommon();
    },
    // 筛选 搜索 公用的方法
    searchCommon() {
      const searchVal = this.searchValue;
      const checkValue = this.checkValue;
      let filterArr = [];
      this.beforeFilterData.forEach((item) => {
        let filterData = [];
        if (checkValue) {
          if (searchVal !== "") {
            filterData = item.children.filter(
              (item2) =>
                item2.name.toLowerCase().indexOf(searchVal.toLowerCase()) >
                  -1 && item2.inStock === true
            );
          } else {
            filterData = item.children.filter(
              (item2) => item2.inStock === true
            );
          }
        } else {
          if (searchVal !== "") {
            filterData = item.children.filter(
              (item2) =>
                item2.name.toLowerCase().indexOf(searchVal.toLowerCase()) > -1
            );
          } else {
            filterData = item.children;
          }
        }
        filterArr.push({
          category: item.category,
          children: filterData,
        });
      });
      this.dataList = filterArr;
      // 如果数组的childre长度都是0，则显示 nodata
      this.noData = filterArr.every((item) => item.children.length === 0);
    },
    // openModal
    openModal(imageUrl) {
      this.imageUrl = "http://" + imageUrl;
      this.visible = true;
    },
    handleOk() {
      this.visible = false;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
ul {
  list-style: none;
}
.app-select {
  width: 90%;
  /* width: 600px; */
  margin: 0 auto;
}
.app-select .ant-checkbox-wrapper {
  text-align: left;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  margin: 10px 0;
}
.app-select .ant-checkbox-input{
  height: 16px;
}
.list-content ul {
  padding: 0;
}
.list-content ul li {
  display: flex;
  justify-content: flex-start;
  flex-direction: column;
}
.list-content ul li h2 {
  text-align: left;
}
.list-content p {
  display: flex;
  justify-content: space-between;
}
.list-content p span{
  flex: 1;
  text-align: left;
}
.list-content p button{
  margin-left: auto;
}
.noData {
  font-size: 14px;
  font-weight: bold;
  text-align: left;
}
.list-content .noStock {
  color: red;
}
.ant-modal-body .img-content {
  display: flex;
  justify-content: center;
  align-items: center;
}
.ant-modal-body .img-content img {
  width: 80%;
}
</style>
