<template>
  <div class="container" @click="popShow = false">
    <van-nav-bar title="天天头条" :fixed="true">
      <template #right id="popShow">
        <van-icon name="search" @click.stop="searchShow = !searchShow" />
      </template>
      <template #left>
        <van-icon name="apps-o" @click.stop="popShow = !popShow" />
        <div class="navbar-links" v-show="popShow">
          <ul>
            <li v-for="(item) in tabs" :key="'links'+item.id">
              <nuxt-link :to="{path:`/${item.code}`}">{{item.name}}</nuxt-link>
            </li>
          </ul>
        </div>
      </template>
    </van-nav-bar>
    <van-search
      v-show="searchShow"
      v-model="searchValue"
      placeholder="please input keywords"
      left-icon
      @search="search"
    >
      <template slot="right-icon">
        <van-icon name="search" @click="search" />
      </template>
    </van-search>

    <div class="main" :class="{'main-searchShow':searchShow}">
      <van-tabs
        :line-width="'0'"
        :border="false"
        :title-inactive-color="'#a2a0a0'"
        :line-height="'1px'"
        :ellipsis="false"
      >
        <van-tab v-for="(item) in tabs" :key="'tabs'+item.id" :name="item.name">
          <template #title>
            <nuxt-link :to="{path:`/${item.code}`}">{{item.name}}</nuxt-link>
          </template>
        </van-tab>
      </van-tabs>
      <van-swipe class="banner" :autoplay="5000" indicator-color="#eee">
        <van-swipe-item v-for="(image, index) in images" :key="index">
          <nuxt-link :to="{path:`/${image.typeCode}/${image.articleId}`}">
            <div class="banner-content">
              <img :src="image.headImgUrl" :alt="image.headImgDesc" />
              <!-- <img v-lazy="image.src" :alt="image.alt" /> -->
              <p class="title">{{image.title}}</p>
            </div>
          </nuxt-link>
        </van-swipe-item>
      </van-swipe>

      <ul class="article-list">
        <nuxt-link
          v-for="(item,key) in f_list"
          :key="key"
          :to="{path:`/${item.typeCode}/${item.articleId}`}"
        >
          <div class="card-item">
            <img :src="item.headImgUrl" :alt="item.headImgDesc" />
            <div class="card-content">
              <h2>{{item.title}}</h2>
              <p class="card-content-footer">
                <span>{{item.gmtCreated}}</span>
                <span>{{item.source}}</span>
              </p>
            </div>
          </div>
        </nuxt-link>
      </ul>
      <div class="pre-next">
        <div class="left">
          <button v-show="page !== 1" @click="pagebe">Prev</button>
        </div>
        <div class="right">
          <button v-show="f_list.length === size" @click="pageNext">Next</button>
        </div>
      </div>
    </div>
    <div class="footer">
      <span style="font-family:arial;">Copyright &copy;2020</span>
      <span @click="mailClick('DMCA')">/&nbsp;DMCA</span>
      <span @click="mailClick('concatUs')">
        /&nbsp;联系我们
      </span>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import { Lazyload } from "vant";

import axios from "axios";

Vue.use(Lazyload);

import homeList from "~/components/home-list.vue";

export default {
  // middleware:'ipCheck',
  components: {
    homeList
  },
  head() {
    return {
      title: "天天头条，更快看到每日头条新闻资讯",
      meta: [
        { charset: "utf-8" },
        {
          hid: "keywords-home",
          name: "keywords",
          content:
            "每日头条新闻,阅读,资讯,热点,头条,社会,娱乐,游戏,财经,军事,科技,健康,历史，综艺，星座，动漫，漫画，搞笑，情感，养生，时尚，体育，国际"
        },
        {
          hid: "description-home",
          name: "description",
          content:
            "天天头条是一个面向全球中文用户的资讯分享网站，每日热点新闻，网罗了国内外热点头条、明星八卦、精彩综艺、体育资讯，内涵段子、福利美图、养生知识、财经，科技前沿等等。它通过智能计算用户的兴趣，让您更加享受阅读的乐趣，畅享高效的阅读平台"
        }
      ],
      link: [{ rel: "icon", type: "image/x-icon", href: "/favicon.ico" }]
    };
  },
  async asyncData() {
    let [banner_data, tabs_data] = await Promise.all([
      axios.get("/apis/api/article/front/recommend"),
      axios.get("/apis/api/articleType/dict")
    ]);
    return {
      images: banner_data.data.data,
      tabs: tabs_data.data.data
    };
  },
  mounted() {
    window.addEventListener("scroll", this.scrollToTop);
  },
  created() {
    this.getList();
  },
  methods: {
    async pagebe() {
      this.page--;
      await this.getList();
      this.backTop();
    },
    async pageNext() {
      this.page++;
      await this.getList();
      this.backTop();
    },
    async getList() {
      let res = await this.$axios.get(
        `/api/article/front/search/${this.page}/${this.size}`
      );
      this.f_list = res.data.data;
    },
    destroyed() {
      window.removeEventListener("scroll", this.scrollToTop);
    },
    search() {
      this.$router.push(`/search-result?wd=${this.searchValue}`);
    },
    async mailClick(type) {
      if (confirm("确定要向wangxiaoqi009@Gmail.com" + "发送邮件么?") == true) {
        parent.location.href =
          "mailto:wangxiaoqi009@Gmail.com" + "?subject=" + type + "";
      }
      let res = await axios.post("/apis/api/event/mail/add");
    },
    backTop() {
      const that = this;
      let timer = setInterval(() => {
        let ispeed = Math.floor(-that.scrollTop / 5);
        document.documentElement.scrollTop = document.body.scrollTop =
          that.scrollTop + ispeed;
        if (that.scrollTop === 0) {
          clearInterval(timer);
        }
      }, 16);
    },
    scrollToTop() {
      const that = this;
      let scrollTop =
        window.pageYOffset ||
        document.documentElement.scrollTop ||
        document.body.scrollTop;
      that.scrollTop = scrollTop;
      // getl
    }
  },
  data() {
    return {
      scrollTop: 0,
      page: 1,
      size: 20,
      f_list: [],
      tabtype: this.activeName,
      searchShow: false,
      searchValue: "",
      popShow: false
      // images:
    };
  }
};
</script>

<style>
.van-nav-bar__title {
  color: #fff;
}
</style>


<style lang="scss" scoped>
@import "~assets/sass/resources.scss";
.container {
  // width: px2rem(375);
  max-width: 500px;
  margin: auto;
  .footer {
    font-size: 13px;
    padding: 20px 13px;
  }
  .pre-next {
    display: flex;
    .left {
      flex: 0 0 50%;
      text-align: left;
      font-size: 16px;
    }
    .right {
      flex: 0 0 50%;
      text-align: right;
      font-size: 16px;
    }
    button {
      border: 1px solid #dad6d6;
      background-color: #fff;
      // padding-top: px2rem(5);
      // padding-left: px2rem(50);
      // padding-right: px2rem(50);
      // padding-bottom: px2rem(5);
      width: 83%;
      height: 40px;
      color: #969696;
      border-radius: 5px;
    }
  }
  .article-list {
    margin-top: 20px;
  }
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
  .van-search {
    padding: 0 13px;
    background-color: transparent;
    margin-bottom: 12px;
    border-radius: 5px;
    position: fixed;
    top: 50px;
    width: 100%;
    z-index: 10;
    max-width: 500px;
    .van-search__content {
      background-color: #f5f8ff;
      box-shadow: 0 0 5px 3px #a2a0a0;
    }
  }
  .main {
    margin-top: 46px;
    padding: 0px 13px 13px 13px;
  }
  .main-searchShow {
    margin-top: 88px;
  }
  .van-nav-bar--fixed {
    max-width: 500px;
    left: unset;
  }
  .van-nav-bar {
    background-color: #161515;
    .van-nav-bar__title {
      color: #fff;
    }
    .navbar-links {
      position: absolute;
      word-break: keep-all;
      background-color: rgba(68, 67, 67, 0.9);
      top: 45px;
      border-radius: 5px;
      height: 250px;
      overflow-y: scroll;
      line-height: 38px;
      color: #fff;
      padding: 10px 26px;
      a {
        color: #fff;
      }
    }
    .van-icon {
      color: #fff;
      vertical-align: middle;
      font-size: 20px;
    }
    // .van-nav-bar__right {
    //   font-size: 17px;
    //   .van-icon-search {
    //     color: #fff;
    //   }
    // }
  }
  .banner {
    height: 200px;
    box-shadow: 0 0 11px 1px #bbbaba;
    border-radius: 5px;
    .banner-content {
      position: relative;
      height: 100%;
      width: 100%;
      img {
        width: 100%;
        height: 100%;
      }
      .title {
        text-align: left;
        padding: 0 20px;
        position: absolute;
        width: 100%;
        left: 0;
        bottom: 0px;
        font-size: 20px;
        font-weight: 400;
        background-color: rgba(0, 0, 0, 0.5);
        color: #fff;
        height: 50px;
        text-overflow: ellipsis;
        white-space: nowrap;
        overflow: hidden;
      }
    }
  }
  .van-tabs {
    margin-bottom: 10px;
    /deep/ .van-tab__text a {
      color: #6d6c6c;
    }
    /deep/ .van-tab--active {
      font-weight: unset;
    }
  }
}
</style>