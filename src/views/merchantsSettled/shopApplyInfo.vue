<template>
  <div class="merchants-settled">
    <cm-header>
      <span slot="left" @click="$router.go(-1)">
        <svg-icon icon-class="green-btn"></svg-icon>
      </span>
      <i>商家入驻</i>
    </cm-header>
    <section class="apply-content">
      <ul class="address-list">
        <ValidationObserver v-slot="{ invalid }" ref="observer" tag="form">
          <li class="address-item">
            <van-cell title="店铺名称" />
            <div class="address-name">
              <ValidationProvider v-slot="{ errors }" :rules="{required:true}">
                <van-field
                  v-model="applyInfo.shopName"
                  :error-message="errors[0]"
                  clearable
                  placeholder="请输入店铺名称"
                />
              </ValidationProvider>
            </div>
          </li>
          <li class="address-item">
            <van-cell title="店铺所在地区" />
            <div class="address-name" @click="showPicker">
              <ValidationProvider v-slot="{ errors }" :rules="{required:true}">
                <van-field
                  v-model="provinceCityDisStr"
                  :error-message="errors[0]"
                  disabled
                  placeholder="请选择省市区"
                />
              </ValidationProvider>
              <div>
                <svg-icon icon-class="arrow"></svg-icon>
              </div>
            </div>
          </li>
          <li class="address-item">
            <van-cell title="详细地址" />
            <div class="address-name">
              <ValidationProvider v-slot="{ errors }" :rules="{required:true}">
                <van-field
                  v-model="applyInfo.address"
                  :error-message="errors[0]"
                  clearable
                  placeholder="如：道路、门牌号、小区、楼栋号、单元室等"
                />
              </ValidationProvider>
            </div>
          </li>
          <li class="address-item">
            <van-cell title="法人姓名" />
            <div class="address-name">
              <ValidationProvider name="regex" v-slot="{ errors }" :rules="{required:true}">
                <van-field
                  v-model="applyInfo.name"
                  clearable
                  :error-message="errors[0]"
                  placeholder="请输入"
                />
              </ValidationProvider>
            </div>
          </li>
          <li class="address-item">
            <van-cell title="法人身份证号码" />
            <div class="address-name">
              <ValidationProvider
                v-slot="{ errors }"
                :rules="{regex:/^(^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$)|(^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])((\d{4})|\d{3}[Xx])$)$/,required:true}"
              >
                <van-field
                  clearable
                  v-model="applyInfo.idCardNo"
                  :error-message="errors[0]"
                  placeholder="请输入"
                />
              </ValidationProvider>
            </div>
          </li>
          <li class="address-item">
            <van-cell title="营业执照注册名称" />
            <div class="address-name">
              <ValidationProvider name="email" v-slot="{ errors }" rules="required">
                <van-field
                  v-model="applyInfo.businessLicenseName"
                  :error-message="errors[0]"
                  placeholder="请输入"
                  clearable
                />
              </ValidationProvider>
            </div>
          </li>
          <li class="address-item">
            <van-cell title="营业执照号码" />
            <div class="address-name">
              <ValidationProvider name="required" v-slot="{ errors }" rules="required">
                <van-field
                  v-model="applyInfo.businessLicenseNo"
                  :error-message="errors[0]"
                  clearable
                  placeholder="请输入"
                />
              </ValidationProvider>
            </div>
          </li>
        </ValidationObserver>
      </ul>
    </section>
    <div class="pay-btn">
      <van-button type="danger" @click="handleSubmitShopInfo" size="large">下一步</van-button>
    </div>
    <van-popup
      v-model="show"
      position="bottom"
      :style="{ height: '40%' }"
      @click-overlay="show = false"
    >
      <div class="address">
        <div class="addressbox">
          <p class="text_btn">
            <span style="float:left;color:#3A3A3A" @click="cancel">取消</span>
            <span style="float:right;color:#EC3924;" @click="complete">完成</span>
          </p>
          <div class="addressSelect">
            <div class="selectbox"></div>
            <ul
              @touchstart="touchStart($event,'province')"
              @touchmove="touchMove($event,'province')"
              @touchend="touchEnd($event,'province')"
              :style="provinceStyle"
              :class="[{'selectAni':addSelect}]"
            >
              <li
                v-for="(item,index) in list"
                :class="[{'addSelectActive':index == provinceIndex}]"
                :key="index"
              >{{item.areaName}}</li>
            </ul>
            <ul
              @touchstart="touchStart($event,'city')"
              @touchmove="touchMove($event,'city')"
              @touchend="touchEnd($event,'city')"
              :style="cityStyle"
              :class="[{'selectAni':addSelect}]"
            >
              <li
                v-for="(item,index) in list2"
                :class="[{'addSelectActive':index == cityIndex}]"
                :key="index"
              >{{item.areaName}}</li>
            </ul>
            <ul
              @touchstart="touchStart($event,'district')"
              @touchmove="touchMove($event,'district')"
              @touchend="touchEnd($event,'district')"
              :style="districtStyle"
              :class="[{'selectAni':addSelect}]"
            >
              <li
                v-for="(item,index) in list3"
                :class="[{'addSelectActive':index == districtIndex}]"
                :key="index"
              >{{item.areaName}}</li>
            </ul>
          </div>
        </div>
      </div>
    </van-popup>
  </div>
</template>

<script>
export default {
  name: "shopApplyInfo",
  data() {
    return {
      isEdit: this.$route.query.isEdit || false,
      systemMessage: {},
      checked: false,
      provinceCityDisStr: "",
      fileList: [],
      idCardNoUrlA: [],
      idCardNoUrlB: [],
      idCardNoUrlC: [],
      businessLicenseUrl: [],
      shopForm: {},
      applyInfo: {},
      parentAreaId: 0,
      mallMessage: {},
      show: false,
      list: [],
      list2: [],
      list3: [],
      provinceStyle: {
        WebkitTransform: "translate3d(0px,0px,0px)"
      },
      cityStyle: {
        WebkitTransform: "translate3d(0px,0px,0px)"
      },
      districtStyle: {
        WebkitTransform: "translate3d(0px,0px,0px)"
      },
      startTop: 0,
      provinceIndex: 0,
      cityIndex: 0,
      districtIndex: 0,
      translateY: 0,
      maxScroll: 0,
      addHeight: 0,
      addSelect: false,
      provinceVal: "",
      cityVal: "",
      areaVal: "",
      val: {
        provinceVal: "",
        cityVal: "",
        areaVal: ""
      }
    };
  },
  created() {
    if (this.isEdit) {
      this.$http.post(`/api/shop/again/display`).then(response => {
        this.applyInfo = response.data.content;
        this.provinceCityDisStr = response.data.content.provinceCityDisStr;
      });
    }
    this.getProvinces();
    this.getCitys();
    this.val.areaVal = {
      name: "",
      value: ""
    };
    // 第一条数据为直辖市 so '-' 符号表示为第三列
    this.list3 = [{ name: "-" }];
  },
  methods: {
    async handleSubmitShopInfo() {
      const isValid = await this.$refs.observer.validate();
      if (isValid) {
        this.applyInfo.isEdit = this.isEdit;
        this.$router.push({
          path: `/merchantsSettled/enterpriseCertification`,
          query: this.applyInfo
        });
      }
    },
    afterReadA(res) {
      let formData = new FormData();
      formData.append("file", res.file);
      this.$http.post(`/api/shop/upload/image`, formData).then(response => {
        this.shopForm.idCardNoUrlA = response.data.content.imageUrl;
      });
    },
    afterReadB(res) {
      let formData = new FormData();
      formData.append("file", res.file);
      this.$http.post(`/api/shop/upload/image`, formData).then(response => {
        this.shopForm.idCardNoUrlB = response.data.content.imageUrl;
      });
    },
    afterReadC(res) {
      let formData = new FormData();
      formData.append("file", res.file);
      this.$http.post(`/api/shop/upload/image`, formData).then(response => {
        this.shopForm.idCardNoUrlC = response.data.content.imageUrl;
      });
    },
    afterReadBusinessLicenseUrl(res) {
      let formData = new FormData();
      formData.append("file", res.file);
      this.$http.post(`/api/shop/upload/image`, formData).then(response => {
        this.shopForm.businessLicenseUrl = response.data.content.imageUrl;
      });
    },
    handleSeveAddresInfo() {
      if (
        !this.applyInfo.shopName ||
        !this.applyInfo.receiverPhone ||
        !this.applyInfo.address ||
        !this.applyInfo.provinceCityDisStr
      ) {
        this.$toast({
          mask: false,
          duration: 1000,
          message: "请输入你的完整的地址信息！"
        });
        return;
      }
      this.$http
        .post(`/api/address/updateUserAddr`, this.applyInfo)
        .then(response => {
          if (response.data.code === 0) {
            this.$router.push("/mine/shippingAddress");
          }
        });
    },

    showPicker() {
      this.show = true;
    },
    // 分层获取中国地址信息
    getProvinces() {
      this.$http
        .get(`/api/address/getCnAreaList?parentAreaId=${this.parentAreaId}`)
        .then(response => {
          this.list = response.data.content;
          this.val.provinceVal = this.list[0];
        });
    },
    getCitys() {
      this.$http
        .get(`/api/address/getCnAreaList?parentAreaId=1`)
        .then(response => {
          this.list2 = response.data.content;
          this.val.cityVal = this.list2[0];
        });
    },

    complete() {
      if (!this.val.areaVal.areaId) {
        this.val.areaVal = {
          areaId: "",
          areaName: ""
        };
      }
      if (!this.val.cityVal.areaId) {
        this.val.cityVal = {
          areaName: "",
          areaId: ""
        };
      }
      this.show = false;
      this.applyInfo.province = this.val.provinceVal.areaId;
      this.applyInfo.city = this.val.cityVal.areaId;
      this.applyInfo.district = this.val.areaVal.areaId;
      this.applyInfo.provinceCityDisStr = this.provinceCityDisStr =
        this.val.provinceVal.areaName +
        this.val.cityVal.areaName +
        this.val.areaVal.areaName;
    },
    cancel() {
      this.show = false;
    },

    // 滑动开始
    touchStart(e, val) {
      e.preventDefault();
      this.addSelect = false;
      this.addHeight = e.currentTarget.children[0].offsetHeight;
      this.maxScroll = this.addHeight * e.currentTarget.children.length;
      this.startTop = e.targetTouches[0].pageY;
      switch (val) {
        case "province":
          this.translateY = parseInt(
            this.provinceStyle.WebkitTransform.slice(
              this.provinceStyle.WebkitTransform.indexOf(",") + 1,
              this.provinceStyle.WebkitTransform.lastIndexOf(",")
            )
          );
          break;
        case "city":
          this.translateY = parseInt(
            this.cityStyle.WebkitTransform.slice(
              this.cityStyle.WebkitTransform.indexOf(",") + 1,
              this.cityStyle.WebkitTransform.lastIndexOf(",")
            )
          );
          break;
        case "district":
          this.translateY = parseInt(
            this.districtStyle.WebkitTransform.slice(
              this.districtStyle.WebkitTransform.indexOf(",") + 1,
              this.districtStyle.WebkitTransform.lastIndexOf(",")
            )
          );
          break;
        default:
          break;
      }
    },
    // 滑动进行中
    touchMove(e, val) {
      e.preventDefault();
      switch (val) {
        case "province":
          if (e.targetTouches[0].pageY - this.startTop + this.translateY > 0) {
            this.provinceStyle.WebkitTransform = "translate3d(0px,0px,0px)";
          } else if (
            e.targetTouches[0].pageY - this.startTop + this.translateY <
            -(this.maxScroll - this.addHeight)
          ) {
            this.provinceStyle.WebkitTransform =
              "translate3d(0px," +
              -(this.maxScroll - this.addHeight) +
              "px,0px)";
          } else {
            this.provinceStyle.WebkitTransform =
              "translate3d(0px," +
              (e.targetTouches[0].pageY - this.startTop + this.translateY) +
              "px,0px)";
          }
          break;
        case "city":
          if (e.targetTouches[0].pageY - this.startTop + this.translateY > 0) {
            this.cityStyle.WebkitTransform = "translate3d(0px,0px,0px)";
          } else if (
            e.targetTouches[0].pageY - this.startTop + this.translateY <
            -(this.maxScroll - this.addHeight)
          ) {
            this.cityStyle.WebkitTransform =
              "translate3d(0px," +
              -(this.maxScroll - this.addHeight) +
              "px,0px)";
          } else {
            this.cityStyle.WebkitTransform =
              "translate3d(0px," +
              (e.targetTouches[0].pageY - this.startTop + this.translateY) +
              "px,0px)";
          }
          break;
        case "district":
          if (e.targetTouches[0].pageY - this.startTop + this.translateY > 0) {
            this.districtStyle.WebkitTransform = "translate3d(0px,0px,0px)";
          } else if (
            e.targetTouches[0].pageY - this.startTop + this.translateY <
            -(this.maxScroll - this.addHeight)
          ) {
            this.districtStyle.WebkitTransform =
              "translate3d(0px," +
              -(this.maxScroll - this.addHeight) +
              "px,0px)";
          } else {
            this.districtStyle.WebkitTransform =
              "translate3d(0px," +
              (e.targetTouches[0].pageY - this.startTop + this.translateY) +
              "px,0px)";
          }
          break;
        default:
          break;
      }
    },
    // 滑动结束
    touchEnd(e, val) {
      e.preventDefault();
      this.addSelect = true;
      switch (val) {
        case "province":
          let provinceTranslateY = parseInt(
            this.provinceStyle.WebkitTransform.slice(
              this.provinceStyle.WebkitTransform.indexOf(",") + 1,
              this.provinceStyle.WebkitTransform.lastIndexOf(",")
            )
          );
          this.provinceIndex = -Math.round(provinceTranslateY / this.addHeight);
          this.provinceStyle.WebkitTransform =
            "translate3d(0px," +
            Math.round(provinceTranslateY / this.addHeight) * this.addHeight +
            "px,0px)";
          this.cityStyle.WebkitTransform = this.districtStyle.WebkitTransform =
            "translate3d(0px,0px,0px)";
          this.cityIndex = this.districtIndex = 0;
          break;
        case "city":
          let cityTranslateY = parseInt(
            this.cityStyle.WebkitTransform.slice(
              this.cityStyle.WebkitTransform.indexOf(",") + 1,
              this.cityStyle.WebkitTransform.lastIndexOf(",")
            )
          );
          this.cityIndex = -Math.round(cityTranslateY / this.addHeight);
          this.cityStyle.WebkitTransform =
            "translate3d(0px," +
            Math.round(cityTranslateY / this.addHeight) * this.addHeight +
            "px,0px)";
          this.districtStyle.WebkitTransform = "translate3d(0px,0px,0px)";
          this.districtIndex = 0;
          break;
        case "district":
          let districtTranslateY = parseInt(
            this.districtStyle.WebkitTransform.slice(
              this.districtStyle.WebkitTransform.indexOf(",") + 1,
              this.districtStyle.WebkitTransform.lastIndexOf(",")
            )
          );
          this.districtIndex = -Math.round(districtTranslateY / this.addHeight);
          this.districtStyle.WebkitTransform =
            "translate3d(0px," +
            Math.round(districtTranslateY / this.addHeight) * this.addHeight +
            "px,0px)";
          break;
        default:
          break;
      }
      // 滑动结束后 处理数据
      this.dataProcessing();
    },
    // 数据处理
    dataProcessing() {
      // 滑动数据传输 数据处理
      this.val.provinceVal = this.list[this.provinceIndex];
      this.provinceVal = this.list[this.provinceIndex].areaId;
      this.val.cityVal = this.list2[this.cityIndex];
      this.cityVal = this.list2[this.cityIndex].areaId;
      this.val.areaVal = this.list3[this.districtIndex];
      this.areaVal = this.list3[this.districtIndex].areaId;
    }
  },
  watch: {
    // 监听省滑动
    provinceVal(value) {
      this.$http
        .get(`/api/address/getCnAreaList?parentAreaId=${value}`)
        .then(res => {
          if (res.data.code === 0) {
            this.list2 =
              res.data.content.length > 0
                ? res.data.content
                : [{ areaName: "-" }];
            if (res.data.content.length < 1) {
              this.list3 = [{ areaName: "-" }];
            }
            // this.cityVal = this.list2[0].areaId;
            this.dataProcessing();
          }
        });
    },
    // 监听市滑动
    cityVal(value) {
      if (value) {
        this.$http
          .get(`/api/address/getCnAreaList?parentAreaId=${value}`)
          .then(res => {
            if (res.data.code === 0) {
              this.list3 =
                res.data.content.length > 0
                  ? res.data.content
                  : [{ areaName: "-" }];
            }
            this.dataProcessing();
          });
      }
    }
  }
};
</script>

<style scoped lang="scss">
.merchants-settled {
  height: 100%;
  padding: 0 16px;
  padding-bottom: 80px;
  .apply-content {
    .address-list {
      padding: 0 20px;
      height: 100%;
      background-color: #fff;
      border-radius: 8px;
      .address-item {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        flex-direction: column;
        /deep/ .van-cell {
          padding-left: 0;
        }
        /deep/ .van-cell__title span {
          font-size: 14px;
          color: #3a3a3a;
        }
        /deep/ .van-cell:not(:last-child)::after {
          border: none;
        }
        .address-name {
          display: flex;
          justify-content: space-between;
          width: 100%;
        }
        .address-tags {
          .isActive {
            color: #e96258 !important;
          }
          /deep/ .van-tag {
            margin-right: 20px;
            width: 40px;
            text-align: center;
          }
        }
      }
    }
  }
  .pay-btn {
    padding: 30px 0;
    /deep/ .van-button--danger {
      background-color: #ec3924;
      line-height: 44px;
      font-size: 18px;
      border-radius: 4px;
    }
  }
  .address .addressbox {
    height: 100%;
    position: absolute;
    z-index: 101;
    width: 100%;
    max-height: 100%;
    overflow: hidden;
    background: #fff;
    bottom: 0px;
  }
  .address .addressbox .text_btn {
    height: 32px;
    font-size: 14px;
    line-height: 32px;
    padding: 0 8px;
    background: #fff;
  }
  .addressSelect .selectbox {
    width: 100%;
    height: 30px;
    border-top: 1px solid #ccc;
    border-bottom: 1px solid #ccc;
    margin-top: 58px;
    background: #fff;
  }
  .address .addressboxbg {
    position: absolute;
    left: 0;
    top: 0;
    z-index: 100;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
  }
  .addressSelect {
    width: 100%;
    position: relative;
    background: #fff;
    height: 190px;
    overflow: hidden;
    -webkit-mask-box-image: linear-gradient(
      0deg,
      transparent,
      transparent 5%,
      #fff 20%,
      #fff 80%,
      transparent 95%,
      transparent
    );
    font-size: 14px;
  }
  .addressSelect ul {
    width: 33.333333%;
    position: absolute;
    left: 0;
    top: 60px;
    -webkit-transform-style: preserve-3d;
    -webkit-backface-visibility: hidden;
    text-align: center;
    padding-left: 0;
  }
  .addressSelect ul li {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: rgba(0, 0, 0, 0.54);
    padding: 3px 0;
  }
  .addressSelect ul:nth-of-type(2) {
    left: 33.333333%;
  }
  .addressSelect ul:nth-of-type(3) {
    left: 66.666666%;
  }
  .addressSelect ul li.addSelectActive {
    color: rgba(0, 0, 0, 0.8);
    font-weight: 600;
    transform: scale(1.3);
    transition: 0.5s;
  }
  .selectAni {
    transition: 0.8s;
  }
}
</style>
