<template>
  <div class="input_wrap">
    <div class="gift_name pc">
      <div>
        <p>
          {{ prize.awards_item }}<span>/{{ prize.awards_points }}點</span>
        </p>
        <div v-if="statusString" class="remind">
          <p>{{ statusString }}</p>
        </div>
      </div>
    </div>
    <!-- <div class="gift_name pc">
      <div>
        <p>{{ prize.awards_item }}</p>
        <p>/{{ prize.awards_points }}點</p>
        <div v-if="statusString" class="remind">
          <p>{{ statusString }}</p>
        </div>
      </div>
    </div> -->
    <!-- <div class="gift_point mb">
      <p>{{ prize.awards_points }}點</p>
    </div> -->
    <div class="group">
      <slot />
    </div>
  </div>
</template>

<script>
// import { getBrowserVersion } from '@/utils';

export default {
  props: {
    prize: {
      type: Object,
      required: true,
    },
  },
  computed: {
    statusString() {
      const amount = this.prize.awards_last_nu;
      if (amount <= 10 && amount > 0) return `*贈品僅剩${amount}個，要換要快!`;
      if (amount === 0) return "*兌換完畢，感謝支持!";
      return false;
    },
  },
  methods: {
    isMobile() {
      let flag = !!window.navigator.userAgent.match(
        /\(i[^;]+;( U;)? CPU.+Mac OS X/
      );
      console.log(flag);
      // console.log(window.navigator.userAgent);
      return flag;
    },
  },
  mounted() {
    // console.log(this.isMobile)
    // console.log(window.navigator.userAgent);
    // if(this.isMobile) {
    //   // console.log("ios");
    //   // alert('ios')
    // }else {
    //   // console.log("pc端");
    //   // alert('pc')
    // }
  },
};
</script>

<style lang="scss">
@import "./ExchangePrizesInput.scss";
</style>
