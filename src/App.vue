<template>
  <div class="page">
    <TheHeader v-if="!$store.state.comingSoon" />

    <!-- <router-view v-slot="{ Component }">
      <transition name="fade" mode="out-in">
        <component :is="Component" :key="$route.path"></component>
      </transition>
    </router-view> -->
    <router-view></router-view>

    <teleport to="#portal-loading">
      <AppLoader v-if="$store.state.loader" />
    </teleport>

    <teleport to="#portal-popup">
      <BrowserBlocker v-if="$store.state.modal === 'browserBlocker'" />
    </teleport>
  </div>
</template>

<script>
import TheHeader from "@/components/TheHeader";
import Modals from "@/components/Modals";
import AppLoader from "@/components/AppLoader";

import { getBrowserVersion } from "@/utils";
import { getTime } from "@/api";
import helper from "@/helpers/App.helper.js";

export default {
  components: {
    TheHeader,
    BrowserBlocker: Modals.BrowserBlockerModal,
    AppLoader,
  },
  async beforeCreate() {
    // 取得活動時間狀態
    const activityTimeResult = await getTime();
    const { result, msg_type, msg, winning } = activityTimeResult;
    // const { result, msg_type, winning } = activityTimeResult;
    this.$store.commit("setWinnerList", winning);
    console.log(result, msg_type);
    if (!result) return;
    if (msg_type === "00") {
      this.$store.commit("setComingSoon");
      this.$router.replace({ name: "coming-soon" });
      return;
    }
    alert(msg);
  },
  async created() {
    const {
      isOldTestStation,
      isOldOfficialStation,
      redirectNewTestStation,
      redirectNewOfficialStation,
      isWebInAppBrowser,
    } = helper;

    // 偵測網址是否正確
    const { host, pathname, hash } = window.location;
    if (isOldTestStation(host)) redirectNewTestStation(pathname, hash);
    if (isOldOfficialStation(host)) redirectNewOfficialStation(pathname, hash);

    // 偵測網頁是否由fb、line、ig的內建瀏覽器開啟
    const currentBrowser = getBrowserVersion().versions;
    // 取得活動時間狀態
    const activityTimeResult = await getTime();
    const { result, msg_type } = activityTimeResult;
    console.log(result, msg_type);
    if (isWebInAppBrowser(currentBrowser) && msg_type !== "00") {
      this.$store.dispatch("showModal", "browserBlocker");
    } else {
      // alert(
      //   '活動時間延長囉！\n「中秋夯什麼 一點抽＆集點換」\n集點&兌換時間延長至2024/10/11(三) 23:59，把握時間快來登錄吧！'
      // );
      // this.isRead = true;
    }

    // 網頁resize畫面大於1200px自動關閉menu
    window.matchMedia("(max-width: 1200px)").addListener((e) => {
      if (e.matches) return;
      this.$store.dispatch("showMenu", false);
    });
  },
  data() {
    return {
      isRead: false,
    };
  },
  methods: {
    initGoogleAuth() {
      const that = this;
      window.gapi.load("auth2", initAuth);

      function initAuth() {
        const googleConfig = helper.getGoogleAuthConfig();
        window.gapi.auth2
          .init(googleConfig)
          .then(successInitAuth, failInitAuth);

        function successInitAuth() {
          that.$store.commit("auth/setGoogleInit");
          that.checkGoogleStatus();
        }

        function failInitAuth(err) {
          console.log(err);
          that.$store.dispatch("showLoader", false);
        }
      }
    },

    checkGoogleStatus() {
      const auth2 = window.gapi.auth2.getAuthInstance();
      this.$store.commit("auth/setAuth2", auth2);
      const googleSignIn = auth2.isSignedIn.get();
      if (googleSignIn) {
        const googleUser = auth2.currentUser.get();
        return this.$store.dispatch("auth/setGoogleAuth", { data: googleUser });
      }
      helper.removeStorageAuthData();
      this.$store.dispatch("showLoader", false);
    },
  },
  mounted() {
    const that = this;
    window.fbAsyncInit = async function () {
      const FB = window.FB;
      const facebookConfig = helper.getFacebookAuthConfig();
      FB.init(facebookConfig);
      FB.AppEvents.logPageView();
      that.$store.commit("auth/setFBInit");
      await FB.getLoginStatus((res) => {
        if (res.status !== "connected") return;
        that.$store.dispatch("auth/setFBAuth", { data: res });
      });
      that.initGoogleAuth();
    };
  },
};
</script>
