<template>
  <view class="container">
    <view class="column money">
      <text>金额</text>
      <input
        :class="['money-input', type]"
        placeholder="0.00"
        :placeholder-class="type"
        name="amount"
        type="digit"
        v-model="statement.amount"
      >
    </view>
    <view class="column" @tap="redirectChoseCategory">
      <text>分类</text>
      <view class="align-right">{{ statement.categoryName }}</view>
    </view>
    <view class="frequent-use" v-if="categories_labels.length > 0">
      <text>猜你想用：</text>
      <text
        class="label"
        v-for="(item, index) in categories_labels"
        :key="index"
        @tap="setCategory(item)"
      >{{ item.name }}</text>
    </view>
    <view class="column" @tap="redirectChoseAsset">
      <text>账户</text>
      <view class="align-right">{{ statement.assetName }}</view>
    </view>
    <view class="frequent-use" v-if="assets_labels.length > 0">
      <text>猜你想用：</text>
      <text
        class="label"
        v-for="(item, index) in assets_labels"
        :key="index"
        @tap="setAsset(item)"
      >{{ item.name }}</text>
    </view>
    <view class="column">
      <text>日期</text>
      <view class="quick-use">
        <text class="label" @tap="quickSetDate('-2')">前天</text>
        <text class="label" @tap="quickSetDate('-1')">昨天</text>
        <text class="label" @tap="quickSetDate('0')">今天</text>
      </view>
      <picker mode="date" start="2017-01-01" @change="dateChange">
        <text>{{ statement.year }}-{{ statement.month }}-{{ statement.day }}</text>
      </picker>
    </view>
    <view class="column location-column">
      <text style="flex: 1">地点</text>
      <view>
        <input name="address" type="text" :value="statement.address">
        <switch :checked="switchCheck" @change="getLocation"/>
      </view>
    </view>
    <view class="column">
      <text>备注</text>
      <input
        class="remark"
        type="text"
        name="description"
        v-model="statement.description"
        placeholder="本笔账单的简单说明"
      >
    </view>
    <view class="btn-save">
      <button v-if="!submiting" @tap="submitStatement">保存</button>
      <button v-else>保存中...</button>
    </view>
  </view>
</template>

<script>
import Util from "@/common/utils/util.js";
import Tools from "@/common/utils/tools";
import { mapMutations, mapActions, mapState } from "vuex";
export default {
  name: "statementEdit",
  props: {
    detail: {
      type: Object,
      default: {
        id: 0,
        type: "expend",
        categoryId: 0,
        assetId: 0,
        categoryName: "请选择分类",
        assetName: "请选择账户",
        amount: "",
        description: "",
        address: "",
        nation: "",
        province: "",
        city: "",
        district: "",
        street: "",
        year: Util.getCurrentYear(),
        month: Util.getCurrentMonth(),
        day: Util.getCurrentDay(),
        time: ""
      },
    },
    type: String,
  },
  data() {
    return {
      statement: {},
      assets_labels: [
        // { id: 1, name: "支付宝" },
        // { id: 2, name: "微信" },
      ],
      categories_labels: [
        // { id: 1, name: "地铁" },
        // { id: 2, name: "一日三餐" },
      ],
      switchCheck: uni.getStorageSync("getLocationSwitch") || false,
    };
  },
  computed: {
    ...mapState({
      submiting: (state) => state.statement.submiting,
    }),
  },
  created() {
    this.statement = this.detail;
    this.statement.type = this.type;
    // console.log(this.statement)
    if (this.statement.id === 0) {
      if (this.switchCheck) this.setLocation();
    }
    this.guessCategory();
    this.guessAsset();
  },
  methods: {
    guessCategory() {},
    guessAsset() {},
    // 分类选择
    redirectChoseCategory() {
      this.$Router.push({
        path: "/pages/bill/chose-category",
        query: {
          type: this.statement.type,
        },
      });
    },
    // 账户选择
    redirectChoseAsset() {
      this.$Router.push({
        path: "/pages/bill/chose-asset",
        query: {
          type: this.statement.type,
        },
      });
    },
    // 赋值分类
    setCategory(category) {
      // console.log(category);
      this.statement.categoryId = category.id;
      this.statement.categoryName = category.name;
    },
    // 赋值账户
    setAsset(asset) {
      // console.log(asset.name);
      this.statement.assetId = asset.id;
      this.statement.assetName = asset.name;
    },
    // 时间改变
    dateChange({ detail }) {
      var date = new Date(detail.value);
      this.statement.year = Util.getCurrentYear(date);
      this.statement.month = Util.getCurrentMonth(date);
      this.statement.day = Util.getCurrentDay(date);
    },
    // 快速选取时间
    quickSetDate(between) {
      const today = new Date();
      let date = today;
      if (between === "-1") {
        date = new Date(today.getTime() - 24 * 60 * 60 * 1000);
      } else if (between === "-2") {
        date = new Date(today.getTime() - 2 * 24 * 60 * 60 * 1000);
      }
      this.statement.year = Util.getCurrentYear(date);
      this.statement.month = Util.getCurrentMonth(date);
      this.statement.day = Util.getCurrentDay(date);
    },
    getLocation(e) {
      // console.log(e)
      let locationSwitch = e.target.value;
      uni.setStorageSync("getLocationSwitch", locationSwitch);
      if (locationSwitch) {
        this.setLocation();
      } else {
        // 关闭获取地理位置
        this.statement.address = "";
        this. nation= "";
        this.province= "";
        this. city= "";
        this.district= "";
        this.street= "";
      }
    },
    async setLocation() {
      await Tools.getLocation().then((res) => {
        let addressComponent = res.regeocodeData.addressComponent;
        this.statement.nation = addressComponent.country;
        this.statement.province = addressComponent.province;
        this.statement.city = addressComponent.city;
        this.statement.district = addressComponent.district;
        this.statement.street = addressComponent.township + addressComponent.streetNumber.street;
        this.statement.address = res.name;
      });
    },
    submitStatement() {
      // console.log("开始提交");
      let statement = this.statement;
      // console.log(statement)
      if (statement.amount === 0 || statement.amount === "" ||  isNaN(statement.amount)) {
        this.$tip.toast("金额不能为零");
        return false;
      }

      if (statement.categoryId === 0) {
        this.$tip.toast("未选择分类");
        return false;
      }

      if (statement.assetId === 0) {
        this.$tip.toast("未选择账户");
        return false;
      }
      if(this.statement.id === 0){
        statement.time = Util.getCurrentTime();
      }
      //触发submit
      this.$emit("submitStatement", statement);
    },
  },
};
</script>

<style lang="scss">
.container {
  > .money {
    height: 100rpx !important;
    line-height: 100rpx !important;
  }
  > .column {
    height: 100rpx;
    line-height: 100rpx;
    padding: 0 24rpx;
    border-bottom: 1px solid #efefef;
    font-size: 28rpx;
    background: white;
    display: flex;
    justify-content: space-between;
    > text {
      min-width: 100rpx;
      &:last-child {
        color: $secondaryText;
      }
    }
    .remark {
      height: 120rpx !important;
      line-height: 120rpx !important;
      text-align: right;
    }
  }
  > .location-column {
    > view {
      display: flex;
      align-items: center;
    }
    input {
      display: inline-block;
      padding-right: 6px;
    }
  }
  .align-right {
    text-align: right;
  }
  .icon-image {
    width: 50rpx;
    height: 50rpx;
  }
  .money-input {
    height: 100rpx;
    font-size: 48rpx;
    text-align: right;
  }
  .frequent-use {
    text-align: right;
    font-size: 24rpx;
    margin: 24rpx;
    > text:first-child {
      color: $secondaryText;
    }
    .label {
      background: #efefef;
      color: $secondaryText;
      padding: 8rpx 16rpx;
      margin-right: 8rpx;
      &:last-child {
        margin-right: 0;
      }
    }
  }
  .quick-use {
    text-align: right;
    font-size: 24rpx;
    > text:first-child {
      color: $secondaryText;
    }
    .label {
      background: #efefef;
      color: $secondaryText;
      padding: 8rpx 16rpx;
      margin-right: 8rpx;
      &:last-child {
        margin-right: 0;
      }
    }
  }
  .input-hidden {
    display: none !important;
  }
  .btn-save {
    margin: 12px;
    button {
      color: white;
      background-color: $primaryColor;
    }
  }
  .frequent-remarks {
    > view {
      display: block;
      font-size: 14px;
      background-color: #efefef;
      margin: 6px 12px;
      padding: 6px 12px;
      color: $secondaryText;
      border-radius: 4px;
    }
  }
}
</style>
