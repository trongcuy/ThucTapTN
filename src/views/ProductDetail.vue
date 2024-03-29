<template>
  <div class="product-detail">
    <div class="product">
      <div class="container">
        <div class="row">
          <div class="col-md-9">
            <div class="row">
              <div class="col-md-6">
                <div class="product__image">
                  <div class="box-thumbnail">
                    <div class="thumbnail-lazy">
                      <img
                        v-if="product.productImage"
                        :src="
                          require(`../assets/images/${product.productImage}`)
                        "
                      />
                    </div>
                  </div>
                  <div class="discount">{{ product.productDiscount }}%</div>
                </div>
              </div>
              <div class="col-md-6">
                <div class="product__name">
                  {{ product.productName }}
                </div>
                <div class="product__price">
                  <div class="old-price">
                    {{ formatMoney(product.productOldPrice) }} <span>đ</span>
                  </div>
                  <div class="price">
                    {{ formatMoney(product.productPrice) }} <span>đ</span>
                  </div>
                </div>
                <div
                  class="product__feature"
                  v-html="product.productFeature"
                ></div>
                <div class="contact">
                  <a
                    class="btn btn--red"
                    href="https://www.messenger.com/t/100005212953356"
                  >
                    Chat Facebook
                  </a>
                  <a href="tel:0988317051" class="btn btn--green">
                    Gọi: 0988317051
                  </a>
                </div>
                <div class="amount">
                  <div class="count">
                    <div @click="decreCount">-</div>
                    <input
                      v-model="currentProduct.count"
                      type="number"
                      min="1"
                    />
                    <div @click="currentProduct.count += 1">+</div>
                  </div>
                  <div class="btn btn--add btn--red" @click="addToCard">
                    Thêm vào giỏ hàng
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="col-md-3">
            <related-products :title="'Máy tính'" />
          </div>
        </div>
      </div>
    </div>
    <div class="infor">
      <div class="container">
        <tabs>
          <tab name="Mô tả" v-html="product.productDescription"></tab>
          <tab name="Thông tin bảo hành" v-html="product.productGuarantee">
          </tab>
        </tabs>
      </div>
    </div>
    <div class="cart" v-if="isCart">
      <div class="title">GIỎ HÀNG</div>
      <div class="list-item">
        <div class="item" v-for="item in cart" :key="item.productId">
          <router-link class="product" to="">
            <div class="product-image">
              <div class="box-thumbnail">
                <div class="thumbnail-lazy">
                  <img
                    :src="require(`../assets/images/${item.productImage}`)"
                  />
                </div>
              </div>
            </div>
            <div class="detail">
              <div class="product-name">
                {{ item.productName }}
              </div>
              <div class="content">
                <span class="count">{{ item.count }} x </span>
                {{ formatMoney(item.productPrice) }}
                <span style="text-decoration: underline">đ</span>
              </div>
            </div>
          </router-link>
          <div class="remove" @click="removeCart(item.productId)">
            <i class="far fa-times-circle"></i>
          </div>
        </div>
      </div>
      <div class="total">
        Tổng cộng: <span class="money">{{ formatMoney(amount) }} </span>
        <span>đ</span>
      </div>
      <div class="close" @click="closeCart">
        <i class="fas fa-times"></i>
      </div>
      <div class="m-b-10">
        <router-link to="/cart" class="btn btn--add w-100 btn--red"
          >Xem giỏ hàng</router-link
        >
      </div>
      <div>
        <router-link to="/pay" class="btn btn--add w-100 btn--red"
          >Thanh toán</router-link
        >
      </div>
    </div>
    <div class="overlay" v-if="isCart"></div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from "vuex";
import RelatedProducts from "../components/related/RelatedProducts.vue";
import { eventBus } from "../main";
export default {
  components: { RelatedProducts },
  props: ["id"],
  data() {
    return {
      isCart: false,
      cart: [],
      currentProduct: {
        productId: "",
        productImage: "",
        productName: "",
        count: 1,
        productPrice: ""
      }
    };
  },
  computed: {
    ...mapGetters("products", { product: "getProduct" }),
    // ...mapGetters("cart", { cart: "getCart" })
    amount() {
      let amount = 0;
      this.cart.forEach(
        item => (amount += item.count * Number(item.productPrice))
      );
      return amount;
    }
  },
  watch: {
    product() {
      this.currentProduct.productImage = this.product.productImage;
      this.currentProduct.productName = this.product.productName;
      this.currentProduct.productPrice = this.product.productPrice;
      this.currentProduct.productId = this.product.productId;
    }
  },
  methods: {
    decreCount() {
      if (this.currentProduct.count > 1) {
        this.currentProduct.count -= 1;
      }
    },
    ...mapActions("products", { loadData: "loadDataById" }),
    // ...mapActions("cart", {
    //   addData: "addToCart",
    //   removeData: "removeCart",
    //   loadCart: "loadCart"
    // }),
    formatMoney(money) {
      return new Intl.NumberFormat().format(money);
    },
    addToCard() {
      // this.addData(this.currentProduct);
      // this.currentCart.push(this.currentProduct);
      let index = this.cart.findIndex(
        item => item.productId == this.currentProduct.productId
      );
      if (index == -1) {
        this.cart.push(this.currentProduct);
      } else {
        this.cart[index].count += 1;
      }
      this.saveCart();
      this.openCart();
      if (this.cart) {
        eventBus.$emit("getCartTotal", this.cart.length);
      }
    },
    removeCart(id) {
      let index = this.cart.findIndex(item => item.productId == id);
      this.cart.splice(index, 1);
      this.saveCart();
      if (this.cart) {
        eventBus.$emit("getCartTotal", this.cart.length);
      }
    },
    saveCart() {
      const parsed = JSON.stringify(this.cart);
      localStorage.setItem("cart", parsed);
    },
    openCart() {
      this.isCart = true;
    },
    closeCart() {
      this.isCart = false;
    }
  },
  created() {
    // this.loadCart();
    this.loadData(this.id);
  },
  mounted() {
    if (localStorage.getItem("cart")) {
      try {
        this.cart = JSON.parse(localStorage.getItem("cart"));
      } catch (error) {
        localStorage.removeItem("cart");
      }
    }
  }
};
</script>

<style lang="scss" scoped>
@import "~@/assets/scss/abstracts/variable.scss";
.product-detail {
  margin-top: 40px;
  .product {
    .col-md-9 {
      .row {
        padding-bottom: 40px;
        border-bottom: 1px solid #ccc;
      }
    }
    &__image {
      position: relative;
      .box-thumbnail {
        padding-top: 80%;
      }
      .discount {
        position: absolute;
        top: 0;
        left: 0;
        z-index: 9;
        width: 50px;
        height: 50px;
        background-color: $color-red;
        color: #fff;
        font-family: Barlow-Bold;
        text-align: center;
        line-height: 50px;
        font-size: 20px;
      }
    }
    &__name {
      font-size: 26px;
      font-family: Barlow-Bold;
      color: #000;
    }
    &__price {
      color: $color-red;
      font-family: Barlow-SBold;
      display: flex;
      font-size: 20px;
      margin: 10px 0;
      span {
        text-decoration: underline;
      }
      .old-price {
        text-decoration: line-through;
        opacity: 0.7;
        margin-right: 20px;
      }
    }
    &__feature {
      margin-left: 30px;
      margin-bottom: 20px;
      ul {
        li {
          list-style-type: disc;
        }
      }
    }
    .contact {
      display: flex;
      justify-content: space-around;
      a {
        font-size: 18px;
      }
    }
    .amount {
      display: flex;
      align-items: center;
      margin-top: 20px;
      .count {
        display: flex;
        margin-right: 30px;
        input {
          width: 40px;
          padding-left: 12px;
          font-size: 16px;
        }
        div {
          border: 1px solid #ccc;
          line-height: 40px;
          width: 30px;
          text-align: center;
          background-color: #f9f9f9;
          cursor: pointer;
        }
      }
    }
  }
  .infor {
    .tabs-component {
      color: #000;
      .tabs-component-tabs {
        color: #000;

        .tabs-component-tab {
          color: #000;

          display: inline-block;
        }
      }
    }
  }
  .cart {
    width: 300px;
    position: fixed;
    background-color: #fff;
    right: 0;
    top: 0;
    height: 100vh;

    z-index: 999;
    padding: 40px 20px;
    .title {
      text-align: center;
      font-family: Barlow-Bold;
      font-size: 20px;
      color: #000;
    }
    .list-item {
      max-height: 480px;
      overflow: auto;
      .item {
        display: flex;
        justify-content: space-between;
        border-bottom: 1px solid #ccc;
        .product {
          width: 85%;
          position: relative;

          padding: 20px 0;
          display: flex;
          justify-content: space-between;
          .product-image {
            width: 30%;
            .box-thumbnail {
              padding-top: 110%;
            }
          }
          .detail {
            width: 55%;
            color: $color-red;
            font-family: Barlow-SBold;
            .content {
              margin-top: 10px;
              opacity: 0.8;
              .count {
                color: $color-black;
              }
            }
          }
          &:hover .product-name {
            color: $color-black;
          }
        }
        .remove {
          padding-top: 20px;
          width: 10%;
          z-index: 999;
          color: $color-black;
          opacity: 0.5;
          font-size: 20px;
          cursor: pointer;
          &:hover {
            opacity: 1;
          }
        }
      }
    }
    .total {
      font-family: Barlow-Bold;
      text-align: center;
      font-size: 18px;
      padding: 20px 0;
      border-bottom: 1px solid #ccc;
      .money {
        color: $color-red;
        text-decoration: none;
      }
      span {
        color: $color-red;
        text-decoration: underline;
      }
    }
    .close {
      position: absolute;
      top: 20px;
      right: 10px;
      font-size: 28px;
      cursor: pointer;
    }
  }
  .overlay {
    z-index: 99;
  }
}
</style>
