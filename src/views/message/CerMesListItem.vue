<template>
  <div>
    <el-card class="box-card1">
      <div slot="header" class="clearfix">
        <span style="font-weight: bold">from:  {{ mesItem.name }} </span>
        <img
          class="messreadimg"
          src="@/assets/处理.svg"
          v-show="this.isRead"
        />
        <img
          class="messreadimg"
          src="@/assets/未处理.svg"
          v-show="!this.isRead"
        />
      </div>
      <div style="margin-bottom: 100px" @click="itemClick" display=false>
        <div class="mes-text">
          邮箱：{{ mesItem.feedback }}
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "CerMesListItem",
  data() {
    return {
      isRead: false,
      dialogFormVisible2: false,
      send_message: {
        to: "",
        text: "",
      }
    };
  },
  inject: ["reload"],
  props: {
    currentIndex: {
      type: Number,
      default: 1,
    },
    mesItem: {
      type: Object,
      default() {
        return {};
      },
    },
    //当前用户id
    userID: {
      type: Number,
      default: 0,
    },
  },
  methods: {
    itemClick() {
      // 将是否已读设置为已读
      // this.sendMessage();
      console.log(this.mesItem);
      
      if (this.mesItem.state=="waiting") {
        const h = this.$createElement;
        this.$msgbox({
          title: "提示",
          message: h("p", null, [h("span", null, "是否要同意申请?")]),
          showCancelButton: true,
          confirmButtonText: "同意",
          cancelButtonText: "拒绝",
          beforeClose: (action, instance, done) => {
            if (action === "confirm") {
              instance.confirmButtonLoading = true;
              instance.confirmButtonText = "执行中...";
              setTimeout(() => {
                // 同意申请
                axios
                  .post("/apply/accept?Aid=" + this.mesItem.id  + "&user="+this.mesItem.user)
                  .then((res) => {
                    console.log(res);
                  })
                  .catch((err) => {
                    console.log(err);
                  });
                this.isRead=true
                history.go(0)
                done();
                setTimeout(() => {
                  instance.confirmButtonLoading = false;
                }, 300);
              }, 1000);
            } else {
              // 拒绝申请
                axios
                  .post("apply/reject?Aid=" + this.mesItem.id  + "&user="+this.mesItem.user)
                  .then((res) => {
                    console.log(res);
                  })
                  .catch((err) => {
                    console.log(err);
                  });
                done();
                this.isRead=true
                setTimeout(() => {
                  instance.confirmButtonLoading = false;
                }, 300);
              history.go(0)
              done();
            }
          },
        })
          .then((action) => {
            this.$message({
              type: "info",
              message: "已同意申请",
            });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已拒绝申请",
            });
          });
      }
      else {
        this.$alert(this.mesItem.state, "消息内容", {
          confirmButtonText: "确定",
          callback: (action) => {
            this.$message({
              type: "info",
              message: `action: ${action}`,
            });
          },
        });
      }
    },
  },
  created: function () {
    if(this.mesItem.state=="waiting"){
      this.isRead=false
    }else{
      this.isRead=true
    }
  },
};
</script>

<style scoped>
.box-card1 {
  width: 95%;
  /* margin-left: 5%; */
  margin-top: 5px;
}

.mesimg {
  width: 50px;
  padding: 10px;
}
.messreadimg {
  position: absolute;
  right: 100px;
  width: 35px;
  padding-top: 0px;
  padding-right: 0px;
}


.mes-text {
  font-size: 16px;
  position: absolute;
  left: 150px;
  padding-left: 100px;
  padding-top: 10px;
  padding-top: 10px;
  padding-right: 20px;
  text-align: left;
}

.mes-time {
  font-size: 16px;
  position: absolute;
  right: 100px;
  padding-left: 0;
  padding-top: 20px;
  padding-bottom: 10px;
  padding-right: 20px;
  overflow: hidden;
  text-align: left;
}
</style>
