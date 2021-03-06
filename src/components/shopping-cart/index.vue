<template>
  <div class="shopping-card">
    <div class="cart" @click="toggle" :class="{ await_pay: totalCount > 0 }">
      <div class="cart-icon">
        <img id="cartIcon" src="./shopping_cart.svg" alt />
        <span class="badge" v-if="totalCount > 0">{{ totalCount }}</span>
      </div>
      <span class="cart-price">￥ {{ totalPrice }}</span>
    </div>
    <div class="desc">另需配送费￥4元</div>
    <div @click="pay" class="minprice" :class="{ highlight: totalPrice >= 20 }">
      <span class="text">{{ actionText }}</span>
    </div>

    <!-- 展开购物车内容 -->
    <transition name="shoppingCartToggle">
      <div class="goods-container" v-show="visible">
        <!-- 遮罩 -->
        <div @click="toggle" class="layer"></div>
        <!-- 购物车列表 -->
        <div class="goods">
          <div class="head">
            <div class="title">购物车</div>
            <div class="clear" @click="clear">清空</div>
          </div>
          <div class="goods-wrapper" ref="goodsWrapper">
            <scroll ref="scroll">
              <ul class="goods-list">
                <li v-for="(item, index) in selectedFoods" :key="index" class="goods-item">
                  <div class="name">{{ item.name }}</div>
                  <span class="price">{{ item.price * item.count }}</span>

                  <div class="action">
                    <food-picker :food-info="item"></food-picker>
                  </div>
                </li>
              </ul>
            </scroll>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>
<script>
import FoodPicker from "@/components/food-picker";
import { log } from "util";

export default {
  name: "shopping-cart",
  components: {
    FoodPicker,
  },
  data() {
    return {
      visible: false,
    };
  },
  props: {
    selectedFoods: {
      type: Array,
      default: () => [{ price: 10, count: 3, name: "food-1" }],
    },
  },
  computed: {
    actionText() {
      return this.totalPrice >= 20
        ? "去结算"
        : this.totalPrice === 0
        ? "￥20元起送"
        : `还差${20 - this.totalPrice}元起送`;
    },
    totalCount() {
      return this.selectedFoods.reduce((prev, current) => {
        return prev + current.count;
      }, 0);
    },
    totalPrice() {
      return this.selectedFoods.reduce((prev, current) => {
        return prev + current.count * current.price;
      }, 0);
    },
  },
  watch: {
    visible(newVal) {
      setTimeout(() => {
        if (newVal) {
          this.$refs.scroll.refresh();
        }
      }, 300);
    },
    totalCount() {
      if (this.totalCount === 0) {
        this.hide();
      }
    },
  },

  methods: {
    async pay() {
      try {
        await this.$alert("支付", "您需要支付" + this.totalPrice + "元", "确定");
      } catch (error) {
        return;
      }
    },
    async clear() {
      try {
        var result = await this.$confirm("清空购物车？");
      } catch (error) {
        return;
      }

      if (result === "confirm") {
        this.$emit("clear");
      }
    },
    toggle() {
      if (this.totalCount === 0) {
        return;
      }
      this.visible = !this.visible;
    },
    show() {
      this.visible = true;
    },
    hide() {
      this.visible = false;
    },
  },
};
</script>
<style lang="less" scoped>
// 购物车切换动画
.shoppingCartToggle-enter-active,
.shoppingCartToggle-leave-active {
  transition: all 0.4s;
  .goods {
    transition: transform 0.4s;
  }
  .layer {
    transition: opacity 0.4s;
  }
}
.shoppingCartToggle-enter,
.shoppingCartToggle-leave-to {
  .goods {
    transform: translateY(100%);
  }
  .layer {
    opacity: 0;
  }
}

.shopping-card {
  display: flex;
  justify-content: space-between;
  max-height: 100px;
  height: 100%;
  background: #07111b;
  align-items: center;
  color: #999;
}
.cart {
  position: relative;
  width: 30%;
  margin-left: 34px;
  white-space: nowrap;
  &.await_pay {
    .cart-icon {
      background: rgb(0, 160, 220);
      color: #fff;
    }
    .cart-price {
      color: #fff;
    }
  }
  .cart-icon {
    display: inline-block;
    z-index: 10030;
    border: 10px solid rgba(7, 17, 27, 1);
    border-radius: 50%;
    position: absolute;
    top: -60px;
    padding: 25px;
    width: 120px;
    height: 120px;
    background: #333;
    margin-right: 4px;

    .badge {
      right: -4px;
      top: -4px;
      text-align: center;
      position: absolute;
      border-radius: 16px;
      width: 50px;
      line-height: 34px;
      background: red;
      color: #fff;
    }
    img {
      width: 50px;
      height: 50px;
    }
  }
  &-price {
    font-size: 32px;
    color: #999;
    font-weight: 800;
    margin-left: 140px;
  }
}
.desc {
  width: 40%;
  text-align: center;
  border-left: 1px solid;
}
.minprice {
  width: 30%;
  background-color: rgba(43, 51, 59, 1);
  height: 100%;
  line-height: 94px;
  text-align: center;
  &.highlight {
    color: #fff;
    background-color: #04b43c;
  }
}
.goods-wrapper {
  max-height: 400px;
  padding: 0 20px;

  background: #fff;
}

.layer {
  z-index: 100;
  background: #07111b;
  opacity: 0.5;
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 90px;
}

.goods-container {
  position: absolute;
  bottom: 97px;
  width: 100%;

  z-index: -1;
}

.goods {
  z-index: 10000;
  position: relative;
  .head {
    padding: 20px;

    background-color: rgba(243, 245, 247, 1);
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 100px;
    border-bottom: 1px solid rgba(7, 17, 27, 0.1);
    .clear {
      color: rgb(0, 160, 220);
    }
  }
  &-item {
    display: flex;
    padding: 20px 4px;
    border-bottom: 1px solid #efefef;
    width: 100%;
    align-items: center;
    justify-content: space-between;

    .price {
      margin-left: auto;
      margin-right: 20px;
      color: red;
      font-size: 28px;
    }
  }
}
</style>
