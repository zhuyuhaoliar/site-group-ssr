<template>
  <van-list
    v-model="loading"
    :finished="finished"
    :loading-text="'Loading'"
    finished-text="No more"
    @load="onLoad"
  >
    <nuxt-link v-for="(item,key) in list" :key="key" :to="{path:`/article/${item.articleId}`}">
      <div class="card-item">
        <div class="card-image">
          <img :src="item.headImgUrl" :alt="item.headImgDesc" />
        </div>
        <div class="card-content">
          <h2>{{item.title}}</h2>
          <p class="card-content-footer">
            <span>{{item.gmtCreated}}</span>
            <span>{{item.source}}</span>
          </p>
        </div>
      </div>
    </nuxt-link>
  </van-list>
</template>

<script>
import Axios from "axios";
export default {
  props: {
    keywords: "",
    date: "",
    source: ""
  },
  data() {
    return {
      page: 0,
      size: 5,
      list: [],
      loading: false,
      finished: false
    };
  },
  watch: {
    keywords() {
      this.page = 1;
      this.finished = false;
      this.list = [];
      this.getList();
    }
  },
  mounted() {},
  methods: {
    onLoad() {
      this.page += 1;
      this.getList();
    },
    async getList() {
      this.loading = true;
      let query = '?'
       query += this.keywords ? `keyword=${this.keywords}&` : "";
       query += this.date ? `date=${this.date}&` : "";
       query += this.source ? `source=${this.source}` : "";
      let res = await this.$axios.get(
        `/api/article/front/search/${this.page}/${this.size}${query}`
      );
      this.list = this.list.concat(res.data.data.contents);
      this.loading = false;
      if (this.list.length >= res.data.data.totalCount) {
        this.finished = true;
      }
    }
  }
};
</script>

<style lang="scss" scoped>
@import "~assets/sass/resources.scss";

.card-item {
  border: 1px solid #eee;
  border-radius: 5px;
  box-shadow: 0px 2px 15px -9px #7b7979;
  margin-bottom: 22px;
  overflow: hidden;
  img {
     font-size: 12px;
    width: 100%;
    display: block;
  }
  .card-content {
    border-top: 1px solid #eee;
    padding: 10px;
    h2 {
      font-size: 18px;
      color: #444;
      font-weight: 600;
    }
    .card-content-footer {
      display: flex;
      font-size: 12px;
      justify-content: space-between;
      margin-top: 11px;
      span {
        color: #aaa;
        background-color: #f5f5f5;
        border: 1px solid #f5f5f5;
        padding: 2px 5px 3px 5px;
        border-radius: 4px;
      }
    }
  }
}
</style>