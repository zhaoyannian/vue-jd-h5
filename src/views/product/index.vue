<template>
  <div class="product-layout">
    <van-swipe :autoplay="3000" :height="350">
      <van-swipe-item v-for="(image, index) in productImages" :key="index">
        <img class="lazy_img" v-if="image" v-lazy="image" />
      </van-swipe-item>
    </van-swipe>

    <span class="btn-left" @click="$router.go(-1)">
      <svg-icon icon-class="green-btn"></svg-icon>
    </span>
    <section v-if="isSpike" class="progress-bar">
      <ul class="progress-left">
        <li class="spike-price">
          <span class="true-price">￥1498.00</span>
          <span class="exchange-rate">1688USDT≈5600CM</span>
        </li>
        <li class="spike-bottom">
          <del class="old-price">￥1498.00</del>
          <progress class="lm-progress" value="22" max="100"></progress>
        </li>
      </ul>
      <ul class="progress-right">
        <div class="right-content">
          <li class="end-time">
            <i>距结束还剩:</i>
          </li>
          <li class="time-value">
            <i>18:26:50</i>
          </li>
        </div>
      </ul>
    </section>
    <ul class="product-content">
      <li class="product-title">
        <div class="text-left">
          <span v-if="detailForm.calculate" class="force-value">{{detailForm.calculate}}算力值</span>
          {{detailForm.productName}}
        </div>
        <span class="heart-full" @click="handleIsLise(detailForm.productId)">
          <svg-icon v-if="isLike" icon-class="heart-full"></svg-icon>
          <svg-icon v-else icon-class="heart-null"></svg-icon>
        </span>
      </li>
      <li class="product-info">
        <b class="product-price">￥{{detailForm.productCnyPrice}}</b>
        <i>邮费：{{detailForm.logisticsDefaultPrice}}</i>
        <i>月销：{{detailForm.monthlySalesQuantity||0}}</i>
      </li>

      <li class="item-info">
        <van-field label="发货地" disabled :placeholder="detailForm.shipAddress">
          <span slot="left-icon" class="anchor-point">
            <svg-icon icon-class="anchor-point"></svg-icon>
          </span>
        </van-field>
      </li>
      <li class="item-info">
        <van-field label="品牌" disabled :placeholder="detailForm.brandName" />
      </li>
      <li class="item-info" @click="handleShowSpecs">
        <van-field label="选择" disabled :placeholder="handleProductSpeces()"></van-field>
        <van-icon name="arrow" />
      </li>
      <li class="store-info">
        <div class="store-detail" @click="handleStoreName">
          <img v-lazy="detailForm.shopLogo" class="store-header" />
          <span class="store-name">{{detailForm.shopName}}</span>
        </div>
        <div class="store-btn">
          <svg-icon icon-class="message-round"></svg-icon>
          <van-button size="small" @click="handleConnectStore" type="danger">进店逛逛</van-button>
        </div>
      </li>
    </ul>
    <div class="item-details">
      <span @click="handleViewDetail">宝贝详情</span>
      <div v-html="detailForm.productDetail" v-show="showDetail" class="html-class"></div>
    </div>
    <van-sku
      @sku-selected="skuSelected"
      v-model="show"
      class="product-sku"
      :sku="sku"
      close-on-click-overlay
      hide-stock
      :goods="goods"
      :goods-id="goodsId"
      @buy-clicked="handleToBuy"
      @add-cart="handleAddToCart"
    >
      <template slot="sku-header-price" slot-scope="props">
        <div class="van-sku__goods-price">
          <span class="van-sku__price-symbol">￥</span>
          <span class="van-sku__price-num">{{ parseFloat((props.price*100).toPrecision(12)) }}</span>
        </div>
        <span class="sku-calculate" v-if="calculate">{{calculate}}倍算力值</span>
        <div class="van-sku-header-item" v-show="stockNum">
          <span class="van-sku__stock" v-if="stockNum">剩余{{stockNum}}件</span>
        </div>
        <div class="van-sku-header-item">
          <span class="van-sku__stock">选择：{{handleProductSpeces()}}</span>
        </div>
      </template>
    </van-sku>
    <div class="product-footer">
      <van-goods-action>
        <van-goods-action-button @click="handleShowSpecs" type="warning" text="加入购物车" />
        <van-goods-action-button type="danger" @click="handleShowSpecs" text="立即购买" />
      </van-goods-action>
    </div>
  </div>
</template>

<script>
export default {
  name: "product",
  data() {
    return {
      show: false,
      calculate: 0,
      stockNum: 0,
      specsName: "",
      showDetail: false,
      detailForm: {},
      skuObj: {},
      isSpike: false, // 是否是秒杀商品 默认是false
      isLike: false, // 是否点赞喜欢
      current: 0,
      stepperValue: "",
      productImages: [],
      // sku:{},
      sku: {
        // 所有sku规格类目与其值的从属关系，比如商品有颜色和尺码两大类规格，颜色下面又有红色和蓝色两个规格值。
        // 可以理解为一个商品可以有多个规格类目，一个规格类目下可以有多个规格值。
        tree: [
          {
            v: [{}]
          }
        ],
        // 所有 sku 的组合列表，如下是：白色1、白色2、天蓝色1、天蓝色2
        list: [{}],
        messages: [],
        price: "0.00",
        stock_num: 0, // 商品总库存
        none_sku: false, // 是否无规格商品
        hide_stock: false // 是否隐藏剩余库存
      },
      goods: {
        // 商品标题
        title: "",
        // 默认商品 sku 缩略图
        picture: ""
      },
      goodsId: ""
    };
  },
  created() {
    this.initData();
  },

  methods: {
    handleIsLise(productId) {
      this.isLike = !this.isLike;
      this.$http.post(`/api/user/addAttention`, {
        id: productId,
        type: 0
      });
    },
    skuSelected({ skuValue, selectedSku, selectedSkuComb }) {
      if (selectedSkuComb) {
        this.specsName = this.specsName + ";" + skuValue.specsName;
        this.calculate = selectedSkuComb.calculate;
        this.stockNum = selectedSkuComb.stock_num;
      } else {
        this.specsName = "";
        this.stockNum = 0;
        this.sku.stock_num = 0;
      }
    },
    onBuyClicked() {},
    handleShowSpecs() {
      this.show = true;
      this.$http
        .get(
          `/api/product/chooseSku?productId=${this.$route.query.productId}&clientType=0`
        )
        .then(response => {
          let responseDataList = response.data.content;
          if (responseDataList.length === 1) {
            this.calculate = responseDataList[0].calculate;
            this.stockNum = responseDataList[0].stock;
          }
          let skuSpecesTree = [];
          // 先获取所有的规格类型的key
          skuSpecesTree = responseDataList[0].speces.map(it => {
            return {
              k: it.specsName,
              k_id: it.specsId,
              v: [],
              k_s: it.specsId
            };
          });
          // 筛选value
          let allSpecesArray = [];
          for (let i = 0; i < responseDataList.length; i++) {
            for (let j = 0; j < responseDataList[i].speces.length; j++) {
              allSpecesArray.push(
                JSON.stringify(responseDataList[i].speces[j])
              );
            }
          }
          let specesArray = [];
          Array.from(new Set(allSpecesArray)).map(it => {
            specesArray.push(JSON.parse(it));
          });
          for (let i = 0; i < skuSpecesTree.length; i++) {
            for (let j = 0; j < specesArray.length; j++) {
              if (skuSpecesTree[i].k_id === specesArray[j].specsId) {
                specesArray[j].name = specesArray[j].specsValue;
                specesArray[j].id = specesArray[j].specsValueId;
                skuSpecesTree[i].v.push(specesArray[j]);
              }
            }
          }
          // 开始筛选list 所有sku的组合列表
          let templeArray = [];
          templeArray = responseDataList.map((it, i) => {
            let listObj = {};
            listObj.stock_num = it.stock;
            listObj.id = it.id;
            listObj.price = it.price;
            listObj.calculate = it.calculate;
            for (let j = 0; j < responseDataList[i].speces.length; j++) {
              listObj[responseDataList[i].speces[j].specsId] =
                responseDataList[i].speces[j].specsValueId;
            }
            return listObj;
          });
          this.sku.tree = skuSpecesTree;
          this.sku.list = templeArray;
        });
    },
    handleProductSpeces() {
      if (this.detailForm.productSpeces) {
        return this.detailForm.productSpeces.join(",");
      }
    },
    initData() {
      this.$http
        .get(
          `/api/product/info?productId=${this.$route.query.productId}&clientType=0`
        )
        .then(response => {
          this.productImages = response.data.content.productImages;
          this.goods.picture = response.data.content.productImages[0];
          this.detailForm = response.data.content;
          this.isLike = Boolean(this.detailForm.attentionFlag);
        });
    },
    handleViewDetail() {
      this.showDetail = true;
    },

    handleAddToCart(skuObj) {
      this.skuObj = skuObj;
      this.$http
        .post(`/api/cart/update`, {
          quantity: this.skuObj.selectedNum,
          skuId: skuObj.selectedSkuComb.id
        })
        .then(response => {
          if (response.data.code === 0) {
            this.$toast.success({
              message: "添加成功~",
              duration: 1500,
              icon: "like-o"
            });
          } else {
            this.$toast.fail({
              message: response.data.msg,
              duration: 1500
            });
          }
        });
    },
    handleToBuy(skuObj) {
      this.skuObj = skuObj;
      this.$router.push({
        path: "/order/confirmOrder",
        query: {
          quantity: this.skuObj.selectedNum,
          skuId: this.skuObj.selectedSkuComb.id
        }
      });
    },
    handleStoreName() {
      this.$router.push("/storeDetail");
    },
    handleConnectStore() {
      this.$router.push({
        path: "/storeDetail",
        query: { merchantInfoId: this.detailForm.merchantShopId }
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.product-layout {
  background-color: white;
  min-height: 812px;
  /deep/ .van-swipe-item {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .lazy_img {
    height: 350px;
    width: 100%;
  }

  .btn-left {
    position: fixed;
    left: 16px;
    top: 14px;
  }

  .progress-bar {
    .progress-left {
      display: inline-block;
      position: relative;
      min-width: 375px;
      height: 50px;
      background: url("../../assets/image/product/rectangle-left.png") no-repeat
        left center;
      background-size: 76% 100%;
      z-index: 2;
      padding: 6px 16px;
      .spike-price {
        display: flex;
        justify-content: flex-start;
        align-items: center;
        color: white;
        .true-price {
          font-size: 17px;
        }
        .exchange-rate {
          font-size: 9px;
          border: 1px solid white;
          border-radius: 6px;
          display: inline-block;
          width: 108px;
          text-align: center;
          line-height: 20px;
          height: 20px;
          margin-left: 10px;
        }
      }
      .spike-bottom {
        display: flex;
        justify-content: flex-start;
        align-items: center;
        .old-price {
          font-size: 11px;
          color: #fff;
          margin-right: 16px;
        }
        .lm-progress {
          width: 70px;
          height: 10px;
          border-radius: 5px;
          color: #ec3924;
          display: inline-block;
        }
      }
    }
    .progress-right {
      display: inline-block;
      position: absolute;
      right: 0;
      width: 100%;
      height: 50px;
      background: url("../../assets/image/product/rectangle-right.png")
        no-repeat right center;
      background-size: 32% 100%;
      padding-right: 16px 20px;
      .right-content {
        display: flex;
        justify-content: flex-end;
        flex-direction: column;
        align-items: flex-end;
        font-size: 10px;
        .end-time {
          padding-top: 10px;
          padding-right: 20px;
          font-size: 10px;
          color: white;
        }
        .time-value {
          padding-right: 10px;
        }
      }
    }
  }
  .product-content {
    padding-top: 20px;
    font-size: 14px;
    .product-title {
      padding-left: 16px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      .text-left {
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
      }
      .force-value {
        display: inline-block;
        vertical-align: middle;
        min-width: 54px;
        text-align: center;
        line-height: 20px;
        height: 20px;
        color: #fff;
        font-size: 9px;
        padding: 0 5px;
        background-color: #ec3924;
        border-radius: 10px 10px;
      }
      .heart-full {
        padding: 0 17px;
      }
      .item-desc {
        font-size: 14px;
        color: #3a3a3a;
        line-height: 20px;
      }
    }
    .product-info {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-left: 16px;
      padding-top: 20px;
      padding-bottom: 10px;
      font-size: 11px;
      margin-right: 16px;
      color: #949497;
      .product-price {
        color: #ec3924;
        font-size: 17px;
      }
    }
    .store-info {
      display: flex;
      justify-content: space-between;
      margin-top: 10px;
      margin-bottom: 10px;
      /deep/ .van-button--danger {
        background-color: #ec3924;
        border: 1px solid #ec3924;
      }
      .store-detail {
        padding-left: 16px;
        padding-top: 16px;
        .store-header {
          display: inline-block;
          width: 24px;
          height: 24px;
          vertical-align: middle;
        }
        .store-name {
          vertical-align: middle;
          color: #3a3a3a;
          padding-left: 4px;
        }
      }
      .store-btn {
        padding-right: 16px;
        padding-top: 10px;
        /deep/ .van-button {
          margin-left: 10px;
        }
      }
    }
    .item-info {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 95%;
      position: relative;
      /deep/ .van-field__left-icon {
        margin-right: 0;
      }
      .anchor-point {
        position: absolute;
        left: 90px;
        .svg-icon {
          width: 15px;
          height: 15px;
        }
      }
    }
    .product-detail {
      padding-left: 16px;
      padding-top: 20px;
      color: #ec3924;
    }
  }
  .item-details {
    text-align: center;
    font-size: 16px;
    color: #3a3a3a;
    span {
      box-shadow: 1px -10px 1px -4px rgba(254, 77, 109, 0.5) inset;
    }
    .html-class {
      margin-top: 20px;
      /deep/ img {
        width: 375px;
      }
      /deep/ div {
        background-size: 50% 100%;
      }
    }
  }
  .product-sku {
    /deep/ .van-sku-actions {
      padding: 0;
    }
    /deep/ .van-button:last-of-type {
      border-radius: 0;
    }
    /deep/ .van-button:first-of-type {
      border-radius: 0;
    }
    /deep/ .van-sku-row__item.van-sku-row__item--active {
      color: #ec3924;
      background: #fff;
      border-color: #ec3924;
    }
    /deep/ .van-sku-row__item {
      border-color: #efeff4;
      background-color: #efeff4;
      color: #3a3a3a;
    }
    /deep/ .van-sku-actions {
      /deep/ .van-button--warning {
        background-color: #f3ca43;
        height: 44px;
        line-height: 44px;
      }
      /deep/ .van-button--danger {
        height: 44px;
        line-height: 44px;
        background-color: #ec3924;
        border: 1px solid #ec3924;
      }
    }
    /deep/ .van-sku-header {
      display: flex;
      justify-content: flex-start;
      align-items: center;
    }
    /deep/ .van-sku-header__goods-info {
      .sku-calculate {
        margin-left: 7px;
        color: white;
        border-radius: 20px 20px;
        background-color: #ec3924;
        display: inline-block;
        font-size: 7px;
        line-height: 17px;
        text-align: center;
        width: 55px;
        height: 17px;
      }
    }
  }
  .select-popup {
    padding: 20px;
    .popup-content {
      .close-icon {
        float: right;
      }
      .popup-top {
        display: flex;
        justify-content: flex-start;
        align-items: flex-start;
        .item-specification {
          display: flex;
          padding-left: 10px;
          justify-content: flex-end;
          align-items: flex-start;
          flex-direction: column;
          font-size: 14px;
          height: 100px;
          color: #3a3a3a;
          .item-count {
            font-size: 14px;
            padding: 3px 0;
          }
          .item-colors {
            font-size: 11px;
            padding: 3px 0;
          }
          .item-price {
            padding: 3px 0;
            color: #ec3924;
            font-size: 17px;
            font-weight: 600;
          }
        }
        img {
          width: 100px;
          height: 100px;
        }
      }
      .popup-center {
        .popup-color {
          .color-text {
            font-size: 14px;
            color: #3a3a3a;
            font-weight: 600;
            padding-bottom: 10px;
          }
          .color-list {
            display: flex;
            flex-direction: row;
            flex-wrap: wrap;
            .color-tag {
              display: flex;
              justify-content: flex-start;
              align-items: center;
              width: 75px;
              height: 29px;
              font-size: 14px;
              background-color: #efeff4;
              border: 1px solid #efeff4;
              border-radius: 4px;
              margin-right: 16px;
              padding-right: 10px;
              margin-top: 10px;
              img {
                padding-right: 10px;
                padding-left: 2px;
                width: 24px;
                height: 24px;
              }
              span {
                min-width: 30px;
              }
            }
            .color-tag.active {
              border: 1px solid #ec3924;
              background-color: white;
              color: #ec3924;
            }
          }
        }
        .popup-size {
          .size-text {
            font-size: 14px;
            color: #3a3a3a;
            font-weight: 600;
            padding-bottom: 10px;
          }
          .size-list {
            display: flex;
            flex-wrap: wrap;
            justify-content: flex-start;
            align-items: center;
            .size-item {
              font-size: 13px;
              width: 42px;
              height: 24px;
              display: flex;
              justify-content: center;
              align-items: center;
              border-radius: 4px;
              background-color: #efeff4;
              margin-right: 16px;
              margin-top: 10px;
            }
          }
        }
        .popup-quantity {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-top: 32px;
          .quantity-text {
            font-weight: 600;
            font-size: 14px;
            color: #3a3a3a;
          }
          /deep/ .van-stepper__input {
            width: 31px;
            height: 20px;
            padding: 0;
            color: #949497;
            font-weight: normal;
            background-color: transparent;
            border: 1px solid #dbdbdb;
          }
          /deep/ .van-stepper__plus {
            border: 1px solid #dbdbdb;
            background-color: transparent;
            width: 16px;
            height: 22px;
            border-radius: 0;
          }
          /deep/ .van-stepper__minus {
            border-radius: 0;
            border: 1px solid #dbdbdb;
            background-color: transparent;
            width: 16px;
            height: 22px;
          }
        }
      }
    }
  }
  /deep/ .van-popup--bottom {
    border-radius: 16px 16px 0 0;
  }
  .product-footer {
    /deep/ .van-button--warning {
      background-color: #f3ca43;
      // border: 1px solid #f3ca43;
      height: 44px;
      line-height: 44px;
    }
    /deep/ .van-goods-action-button--first {
      border-radius: 0;
      margin: 0;
    }
    /deep/ .van-goods-action-button--last {
      border-radius: 0;
      margin: 0;
    }
    /deep/ .van-button--danger {
      height: 44px;
      line-height: 44px;
      background-color: #ec3924;
      border: 1px solid #ec3924;
    }
  }
}
</style>
