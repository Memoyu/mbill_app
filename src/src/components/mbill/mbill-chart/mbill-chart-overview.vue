<template>
  <view class="overview">
    <view class="overview-header">
      <view class="overview-header__card">
        <view class="expend amount">{{ header.monthExpend }}</view>
        <view class="text-small">总支出</view>
      </view>
      <view class="overview-header__card">
        <view class="income amount">{{ header.monthIcome }}</view>
        <view class="text-small">总收入</view>
      </view>
      <view class="overview-header__card">
        <view class="transfer amount">{{ header.monthTransfer }}</view>
        <view class="text-small">转账</view>
      </view>
      <view class="overview-header__card">
        <view class="expend amount">{{ header.monthRepayment }}</view>
        <view class="text-small">还款</view>
      </view>
    </view>
    <view class="overview__count-result">
      <text class="fs12">总计</text>（收入 - 支出 - 还款）：
      <text :class="[header.total < 0 ? 'expend' : 'income']">{{ header.total }}</text>
    </view>
    <view class="overview__statements">
      <view v-for="(item, index) in statements" :key="index">
        <mbill-bill-statement-item :bill="item"></mbill-bill-statement-item>
      </view>
      <core-empty v-if="statements.length <= 0" :title="emptyTitle"></core-empty>
      <uni-load-more v-else-if="showLoadMore" :status="status" :content-text="contentText"></uni-load-more>
    </view>
  </view>
</template>

<script>
import { mapActions, mapState } from "vuex";
import Value from "@/common/utils/value";

export default {
  name: "overviewChart",
  props: {
    date: {
      type: Object
    }
  },
  data() {
    return {
      header: {
        monthExpend: 0.0,
        monthIcome: 0.0,
        monthTransfer: 0.0,
        monthRepayment: 0.0,
        total: 0.0
      },
      statements: [],
      categories: [],
      total: 0,
      page: {
        UserId: 0,
        Year: 0,
        Month: 0,
        Size: 10,
        Page: 0
      },
      emptyTitle: "空空如也！",
      showLoadMore: false,
      status: "more",
      contentText: {
        contentdown: "上拉加载更多",
        contentrefresh: "加载中",
        contentnomore: "没有更多"
      }
    };
  },
  computed: {
    ...mapState({
      userInfo: state => state.user.userInfo
    })
  },
  created() {
    this.initData();
  },
  watch: {
    date() {
      this.initData();
    }
  },
  methods: {
    ...mapActions(["getPagesAsync", "getTotalAsync"]),
    initData() {
      this.getStatementTotal();
      this.getStatementList(true);
    },
    onLoadMore() {
      this.status = "more";
      this.showLoadMore = true;
      var totalPage = this.total / this.page.Size;
      var currentPage = this.page.Page + 1;
      if (currentPage > totalPage) {
        this.status = "noMore";
        return;
      }

      this.page.Page += 1;
      this.status = "loading";
      this.getStatementList();
    },
    // 获取当各类账单总计
    async getStatementTotal() {
      let that = this;
      let res = await that.getTotalAsync({
        UserId: that.userInfo.id,
        Year: that.date.year,
        Month: that.date.month
      });
      that.header = res;
      that.header.total =
        Value.returnFloat(that.header.monthIcome -
        that.header.monthExpend -
        that.header.monthRepayment);
    },
    // 获取账单列表
    async getStatementList(isCover = false) {
      let that = this;
      that.page.UserId = that.userInfo.id;
      that.page.Year = that.date.year;
      that.page.Month = that.date.month;
      let res = await that.getPagesAsync(that.page);
      if (isCover) {
        uni.pageScrollTo({ scrollTop: 0, duration: 0 });
        that.statements = res.items;
      } else {
        that.statements = [...that.statements, ...res.items];
      }
      that.total = res.total;
    },
  }
};
</script>

<style lang="scss">
.overview {
  .overview-header {
    padding: 24rpx 0;
    border-bottom: 1px solid #f4f4f4;
    > .overview-header__card {
      display: inline-block;
      width: 25%;
      text-align: center;
      .text-small {
        font-weight: bold;
        font-size: 24rpx;
      }
      .amount {
        font-size: 32rpx;
      }
    }
  }
  .overview__count-result {
    padding: 16rpx 24rpx;
    font-weight: bold;
    font-size: 24rpx;
  }
  .overview__category-chart {
    margin: 12px;
    padding: 12px;
    box-shadow: 0 0 4px #eee;
    .budget-chart-item {
      display: inline-block;
      font-size: 24rpx;
      text-align: center;
    }
  }
  .overview__statements {
    margin: 12px;
    padding: 12px;
    box-shadow: 0 0 4px #eee;
  }
}
</style>
