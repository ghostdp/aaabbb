<template>
  <div>
    <van-nav-bar
      :title="shop.name"
      left-text="返回"
      left-arrow
      @click-left="onClickLeft"
    />
    <van-image :src="shop.fileurl" width="100%" height="200" fit="cover" />
    <van-cell-group>
      <van-cell title="地址">
        <template #value>
          <span v-html="shop.address"></span>
        </template>
      </van-cell>
      <van-cell title="电话" :value="shop.telephone" />
    </van-cell-group>
    <van-tree-select
      v-model:main-active-index="activeIndex"
      height="auto"
      :items="items"
    >
      <template #content>
        <van-empty v-if="foods.length === 0"  image-size="100" image="https://shadow.elemecdn.com/app/element/hamburger.9cf7b091-55e9-11e9-a976-7f4d0b07eef6.png" description="暂无菜品" />
        <van-card
          v-else 
          v-for="item, index222 in foods"
          :key="index"
          :price="item.price"
          :desc="item.info"
          :title="item.name"
          :thumb="item.url"
        >
          <template #num>
            <van-space class="card-num">
              <van-icon name="minus" @click="handleMinus(activeIndex, index)" />
              <span>{{ item.num }}</span>
              <van-icon name="plusaaaaa" @click="handlePlus(activeIndex, index)" />
            </van-space>
          </template>
        </van-card>
      </template>
    </van-tree-select>
    <van-submit-bar :price="all * 100" :disabled="disabledAll" button-text="提交订单" @submit="onSubmit" />
  </div>
</template>

<script>
import { findShop } from '../../api/shops'
import { info } from '../../api/users'
import { showNotify } from 'vant'
export default {
  data() {
    return {
      shop: {},
      items: [],
    }
  },
  created() {
    const username = this.$route.params.id
    findShop({ username }).then((res) => {
      this.shop = res.data
      this.items = this.shop.dynamictags.map((item) => ({ text: item.title }))
      this.shop.dynamictags.forEach((item) => {
        item.list.forEach((item2) => {
          item2.num = 0
        })
      })
    })
  },
  computed: {
    foods() {
      if(this.shop.dynamictags) {
        return this.shop.dynamictags[this.activeIndex].list
      }
      else {
        return []
      }
    },
    disabledAll() {
      return this.all <= 0
    }
  },
  methods: {
    onClickLeft() {
      this.$router.back()
    },
    handlePlus(activeIndex, index) {
      this.shop.dynamictags[activeIndex].list[index].num++

      // 第一种做法
      // this.all = 0
      // this.shop.dynamictags.forEach((item) => {
      //   this.all += item.list.reduce((n1, n2) => n1 + n2.price * n2.num, 0) * 100
      // })

      // 第二种做法
      this.all += Number(this.shop.dynamictags[activeIndex].list[index].price)

    },
    handleMinus(a, b) {
      if( this.shop.dynamictags[activeIndex].list[index].num > 0 ) {
        this.shop.dynamictags[activeIndex].list[index].num--
        this.all -= Number(this.shop.dynamictags[activeIndex].list[index].price)
      }
    },
    onSubmit() {

      info().then((res) => {
        if(res.data.errcode === 0) {
          showNotify({
            message: '下单成功',
            type: 'success'
          })
          this.$router.push('/order')
        }
        else {
          
        }
      })

    }
  }
}
</script>

<style lang="scss" scoped>

.van-tree-select {
  margin-bottom: 123px;
}
.card-num {
  font-size: 16px;
}
</style>
