<template>
  <div class="container">
    <div class="userinfo">
      <img :src="userinfo.avatarUrl" alt>
      <p v-if="userinfo.openId">{{userinfo.nickName}}</p>
      <button v-else open-type="getUserInfo" @getuserinfo="getUserInfo">点击登录</button>
    </div>
    <YearProgress></YearProgress>
    <button v-if="userinfo.openId" @click="scanbook" class="btn">添加图书</button>
  </div>
</template>
<script>
import { post, showSuccess,showModal} from "../../util";
import qcloud from "wafer2-client-sdk";
import config from "../../config";
import YearProgress from "../../components/YearProgress";
export default {
  components: {
    YearProgress

  },
  data() {
    return {
      userinfo: {
        avatarUrl: "../../static/img/book.png",
        nickName: "点击登录"
      }
    };
  },
  methods: {
    async  addbook(isbn){
        const res = await post('/weapp/addbook',{
            isbn,
            openid:this.userinfo.openId
        })
        showModal('添加成功',`${res.title}添加成功`)
      },
    scanbook() {
      console.log("添加图书")
      wx.scanCode({
        success: res => {
          if(res.result){
              this.addbook(res.result)
          }
        }
      });
    },
    login() {
      qcloud.setLoginUrl(config.loginUrl);
      const session = qcloud.Session.get();
      console.log("session", session);
      if (session) {
        console.log("二次登录");
        // 第二次登录
        // 或者本地已经有登录态
        // 可使用本函数更新登录态
        qcloud.loginWithCode({
          success: userinfo => {
            wx.setStorageSync("userinfo", userinfo);
            this.userinfo = wx.getStorageSync("userinfo");
            console.log(this.userinfo);
            showSuccess('登陆成功');
          },
          fail: err => {
            console.error("222", err);
          }
        });
      } else {
        console.log('首次登陆');
        qcloud.login({
          success: userinfo => {
            wx.setStorageSync("userinfo", userinfo);
            this.userinfo = wx.getStorageSync("userinfo");
            console.log(this.userinfo);
            showSuccess('登陆成功');
          },
          fail: err => {
            console.log(err);
          }
        });
      }
    },
    getUserInfo(e) {
      if (e.mp.detail.errMsg != "getUserInfo:ok") {
        console.log('授权失败');
      } else {
        console.log('授权成功');
        this.login();
      }
    }
  },
  created() {
    if (wx.getStorageSync("userinfo").openId) {
      this.login();
    }
  }
};
</script>
<style lang="scss">
.container {
  padding: 0 30rpx;
}
.userinfo {
  margin-top: 100rpx;
  text-align: center;
  img {
    width: 128rpx;
    height: 128rpx;
    margin: 20rpx;
    border-radius: 50%;
  }
}
</style>