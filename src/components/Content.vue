<template>
  <div class="Content" :style="width">
    <!-- 面包屑盒子 -->
    <el-row>
      <!-- 面包屑 -->
      <el-col class="left" :span="12">
        <el-breadcrumb>
          <el-breadcrumb-item v-for="(v,i) in crumbsArr" :key="i" :to="{path:v.path}">{{v.title}}</el-breadcrumb-item>
        </el-breadcrumb>
      </el-col>

      <!-- 个人名片 -->
      <el-col class="pirce" :span="12">
        <!-- 下拉菜单 -->
        <el-dropdown @command="handleCommand">
          <span class="el-dropdown-link">
            <span ref="text">{{user}}</span>
            <i class="el-icon-arrow-down el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="a">个人中心</el-dropdown-item>
            <el-dropdown-item command="b">退出系统</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <!-- 头像 -->

        <el-avatar :size="50" :src="imgUrl" style="margin-left:10px"></el-avatar>
      </el-col>
    </el-row>
    <!-- 主要内容出口 -->
    <router-view class="box"></router-view>
  </div>
</template>

<script>
// 引入local
import local from "@/utils/local";
// ajax
import { accountinfo } from "@/api/account";
export default {
  data() {
    return {
      // 面包屑数组
      crumbsArr: [],
      user: "", //用户名
      imgUrl: "", //头像
      // 动态宽度
      width: {
        width: ""
      }
    };
  },
  methods: {
    // 下拉菜单触发事件
    handleCommand(command) {
      if (command === "a") {
        // 点击跳转到个人中心
        this.$router.push("/home/acc/personal");
      } else if (command === "b") {
        // 退出系统
        this.$message({ message: "欢迎下次再来", type: "success" });
        local.clear(); // 清除本地
        this.$router.push("/login"); // 跳转到登录
        location.reload();
      }
    },
    // 面包屑
    routerChange() {
      let arr = [{ title: "首页", path: "/home" }];
      // 遍历 $route.matched 这个数组
      this.$route.matched.forEach(item => {
        // 当每一项里有值 且 有title 标题时
        if (item.meta && item.meta.title) {
          arr.push({
            title: item.meta.title,
            path: item.path
          });
        }
      });
      // 赋值给 存放面包屑数组
      this.crumbsArr = arr;
    },

    //  更新个人信息
    async accInfo() {
      let user = await accountinfo();
      // 更新个人信息
      this.user = user.account;
      this.imgUrl = user.imgUrl;
      // 把数据存到本地
      local.set("user", user);
    },

    // 定义一个函数随机颜色
    color() {
      let arr = ["#1abc9c", "#f1c40f", "#9b59b6", "#f39c12", "#51627a"];
      if (this.$refs.text) {
        let a = setInterval(() => {
          let i = Math.floor(Math.random() * arr.length);
          this.$refs.text.style.color = arr[i];
        }, 500);
      } else {
        clearInterval(a);
      }
    }
  },
  created() {
    // 面包屑
    this.routerChange();
    // 更新信息
    this.accInfo();

    // console.log(this.accInfo());

    // 接受个人中心的通知
    this.$bus.$on("updata_avatar", () => {
      this.accInfo(); //重新获取数据
    });
  },
  watch: {
    "$route.path"() {
      this.routerChange();
    }
  },
  mounted() {
    // 监听浏览器的宽度
    window.onresize = () => {
      return (() => {
        this.width.width = document.body.clientWidth - 360 + "px";
        // console.log(this.width.width);
      })();
    };

    this.color();
  }
};
</script>

<style lang="less" scoped>
.Content {
  display: flex;
  flex-direction: column;
  .el-row {
    flex: 0 0 60px;
    display: flex;
    align-items: center;
    padding: 0px 70px 0px 20px;
    box-sizing: border-box;
    .pirce {
      display: flex;
      align-items: center;
      justify-content: flex-end;

      .el-col {
        width: 50px;
        margin-left: 20px;
      }

      .el-dropdown-link {
        font-weight: 600;
        // letter-spacing: 1px;
        font-size: 18px;
      }
    }
  }
  .box {
    flex: 1;
    background-color: rgb(232, 234, 235);
    overflow-y: scroll;
    padding: 20px;
    box-sizing: border-box;
  }
}
</style>
