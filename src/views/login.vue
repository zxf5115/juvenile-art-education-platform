<template>
  <div>
    <el-container id="auth-container">
      <el-main style="position: relative;">
        <div id="login-box">
          <div class="header">
            {{ this.website_chinese_title }}
            <p>{{ this.website_english_title }}</p>
          </div>
          <div class="main" style="width: 100%;">
            <el-form ref="form" :model="dataForm" :rules="dataRule">
              <el-form-item prop="username">
                <el-input v-model="dataForm.username" :placeholder="$t('login.username')" class="cuborder-radius"/>
              </el-form-item>
              <el-form-item prop="password">
                <el-input v-model="dataForm.password" type="password" :placeholder="$t('login.password')" class="cuborder-radius"/>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="login" class="submit-btn">
                  {{ $t('common.login') }}
                </el-button>
              </el-form-item>
            </el-form>
          </div>
        </div>
        <div class="copyright" v-html="this.copyright"></div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
  export default {
    name: "login",
    data() {
      return {
        dataForm:
        {
          username: '',
          password: ''
        },
        dataRule:
        {
          username: [
            { required: true, message: this.$t('login.rules.username.require'), trigger: 'blur' },
          ],
          password: [
            { required: true, message: this.$t('login.rules.password.require'), trigger: 'blur' },
          ],
        }
      };
    },
    methods: {
      get_system_info:function() {
        this.$http({
          url: this.$http.adornUrl(`/kernel`),
          method: 'post',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.status === 200) {

            this.website_logo          = data.data.logo
            this.website_chinese_title = data.data.web_chinese_name
            this.website_english_title = data.data.web_english_name
            this.website_description   = data.data.description
            this.copyright             = data.data.copyright
          }
        })
      },
      // 登录
      login: function() {
        var _this = this;

        this.$http({
          url: '/api/login',
          method: 'post',
          data: this.$http.adornData({
            'username': this.dataForm.username,
            'password': this.dataForm.password,
          })
        }).then(({data}) => {
          if (data && data.status === 200) {
            // 存储用户的token
            localStorage.setItem("token", data.data.token);
            localStorage.setItem('user_info',JSON.stringify(data.data.user_info));

            _this.$message({ message: _this.$t('login.login_success'), type: "success" });

            let is_new = data.data.user_info.is_new;
            let id = data.data.user_info.id;

            if(1 == is_new)
            {
              _this.$router.push({ name: "user_password_form", query: {id: id} });
            }
            else
            {
              _this.$router.push({ name: "home" });
            }
          }
          else {
            _this.$message.error(data.message);
            return;
          }
        })
      },
      check_user_login: function() {
        var _this = this;

        _this.$http({
          url: _this.$http.adornUrl(`/check_user_login`),
          method: 'get',
          params: _this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.status === 200) {
            _this.$router.push({ name: "home" });
          }
          else {
            _this.$router.push({ name: "login" });
          }
        })
      },
    },
    computed: {
      website_logo: {
        get () { return this.$store.state.system.website_logo },
        set (val) { this.$store.commit('system/updateWebsiteLogo', val) }
      },
      website_chinese_title: {
        get () { return this.$store.state.system.website_chinese_title },
        set (val) { this.$store.commit('system/updateWebsiteChineseTitle', val) }
      },
      website_english_title: {
        get () { return this.$store.state.system.website_english_title },
        set (val) { this.$store.commit('system/updateWebsiteEnglishTitle', val) }
      },
      website_description: {
        get () { return this.$store.state.system.website_description },
        set (val) { this.$store.commit('system/updateWebsiteDescription', val) }
      },
      copyright: {
        get () { return this.$store.state.system.copyright },
        set (val) { this.$store.commit('system/updateCopyright', val) }
      }
    },
    created: function() {
      this.get_system_info();
      // this.check_user_login();
    }
  };
</script>

<style scoped>
  #auth-container {
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
  background-color: #F6F8FB;
}

#logo-name {
  width: 200px;
  height: 38px;
  font-size: 30px;
  font-family: Times New Roman, Georgia, Serif;
  color: #2196f3;
  margin-left: 20px;
}

#login-box {
  position: absolute;
  width: 350px;
  min-height: 380px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  border-radius: 5px;
  box-shadow: 0 0 0 #ccc;
  box-shadow: 0 4px 14px 0 rgba(206, 207, 209, .5);
  padding: 10px 20px;
}

#login-box .header {
  width: 100%;
  height: 90px;
  font-size: 22px;
  margin: 25px 0 20px 0;
  text-align: center;
}
#login-box .header p {
  font-size: 14px;
  margin-top: 10px;
}

#login-box .submit-btn {
  width: 100%;
  height: 38px;
  border-radius: 2px;
}

.send-code-btn {
  width: 140px;
  height: 40px;
  line-height: 40px;
  display: inline-block;
  background: #f3ecec;
  text-align: center;
  color: #777373;
  cursor: pointer;
  user-select: none;
}

.send-code-btn:active {
  background: #e4dbdb;
}

.send-sms-disable {
  cursor: not-allowed !important;
  background: #f7f7f7 !important;
  color: silver !important;
}

.cuborder-radius>>>.el-input__inner {
  border-radius: 1px !important;
  height: 38px;
}

.links{
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 400;
}


.preview-account {
  text-align: center;
}

.preview-account p {
  height: 25px;
  line-height: 25px;
  color: rgb(45, 44, 44);
  font-weight: 100;
  font-size: 12px;
}

.copyright {
  position: absolute;
  bottom: 30px;
  left: 0;
  right: 0;
  width: 70%;
  text-align: center;
  margin: 0 auto;
  font-size: 12px;
  color: #b1a0a0;
}

.copyright a {
  color: #777272;
  font-weight: 400;
}

.login-broadside {
  padding: 0;
  background-image: url(data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiB3aWR0aD0iNDc5IiBoZWlnaHQ9IjkwMCI+PGRlZnM+PGxpbmVhckdyYWRpZW50IGlkPSJjIiB4MT0iMCUiIHkxPSI1MCUiIHkyPSI1MCUiPjxzdG9wIG9mZnNldD0iMCUiIHN0b3AtY29sb3I9IiMwMDY5RkYiLz48c3RvcCBvZmZzZXQ9IjEwMCUiIHN0b3AtY29sb3I9IiMwMDY5RkYiLz48L2xpbmVhckdyYWRpZW50PjxwYXRoIGlkPSJhIiBkPSJNMCAwaDQ3OXY5MDBIMHoiLz48L2RlZnM+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj48bWFzayBpZD0iYiIgZmlsbD0iI2ZmZiI+PHVzZSB4bGluazpocmVmPSIjYSIvPjwvbWFzaz48dXNlIGZpbGw9IiMyMzVCREEiIHhsaW5rOmhyZWY9IiNhIi8+PGcgc3Ryb2tlPSIjMzI3MEZGIiBzdHJva2UtbGluZWpvaW49InJvdW5kIiBzdHJva2Utd2lkdGg9IjEwIiBtYXNrPSJ1cmwoI2IpIiBvcGFjaXR5PSIuMyI+PHBhdGggZD0iTTc4MS44MjYgMTQ2Ljg4NkwzMDguODkxLTIwOS40OTVsLTcyLjE2NyA1ODcuNzYzeiIvPjxwYXRoIGQ9Ik0zMzMuMjY0LTE1Mi42OThMMjc0LjA5IDMyOS4yMjcgNzIxLjAzNyAxMzkuNTF6Ii8+PHBhdGggZD0iTTM1Ni4yNjEtOTQuODY0bC00Ni4wNSAzNzUuMDYgMzQ3LjgzNS0xNDcuNjQ5eiIvPjxwYXRoIGQ9Ik0zODAuNjMzLTM4LjA2NmwtMzMuMDU3IDI2OS4yMiAyNDkuNjgtMTA1Ljk4M3oiLz48cGF0aCBkPSJNNDA0LjAwNSAxOC43MzFsLTIwLjA2IDE2My4zODIgMTUxLjUyMy02NC4zMTh6Ii8+PHBhdGggZD0iTTQ3My44OCAxMTEuMDA2bC00Ny4xMjgtMzUuNTE0LTcuMTkxIDU4LjU3MXoiLz48L2c+PHBhdGggZmlsbD0idXJsKCNjKSIgZD0iTTg0LjM2MSAzOS42MjFsLTgyLjY1NS45NDNhMS4yOCAxLjI4IDAgMDAtLjc2Ni4xNzYgMS4zMTkgMS4zMTkgMCAwMC0uNDU1IDEuNzk2bDQyLjE5IDcxLjkxMmMuMDEzLjAyMi4wMzIuMDM3LjA0Ni4wNTguMDQzLjA2Ny4wOTEuMTI5LjE0NC4xODUuMDEuMDEuMDE5LjAyMy4wMy4wMzNhMS4yODYgMS4yODYgMCAwMDEuNTY1LjE5NmMuMjA1LS4xMjEuMzY1LS4yOS40NzQtLjQ4Ny4wMi0uMDM1LjAyOS0uMDc0LjA0NC0uMTExbDQwLjUyOC03Mi43NDhhMS4zMjIgMS4zMjIgMCAwMC0uMDE0LTEuMzEgMS4yOTQgMS4yOTQgMCAwMC0xLjExNi0uNjQzaC0uMDE1IiBtYXNrPSJ1cmwoI2IpIi8+PHBhdGggZmlsbD0iIzE0QzBGRiIgZD0iTS0xNS4zMjUtMzdMLTUyIDI2LjQ1bDM2LjY3NSA2My40NWg3My4zNUw5NC43IDI2LjQ1IDU4LjAyNS0zN3oiIG1hc2s9InVybCgjYikiIG9wYWNpdHk9Ii4xOTQiLz48ZyBzdHJva2U9IiMzMjcwRkYiIHN0cm9rZS1saW5lam9pbj0icm91bmQiIHN0cm9rZS13aWR0aD0iMTAiIG1hc2s9InVybCgjYikiIG9wYWNpdHk9Ii4zIj48cGF0aCBkPSJNMjQwLjE3MiA4OTkuOTU3bC0zMzMuNjg1LTEyMS40NSA2MS42NjMgMzQ5LjcwNHoiLz48cGF0aCBkPSJNLTY5LjM3MSA4MDcuMDcybDUwLjUxNiAyODYuNDkzTDIwMy45OTggOTA2LjU3eiIvPjxwYXRoIGQ9Ik0tNDYuMjMgODM1LjYzOGwzOS4zNzIgMjIzLjI4MiAxNzMuNjgtMTQ1LjczN3oiLz48cGF0aCBkPSJNLTIyLjA4OCA4NjQuMjA0bDI4LjIyNCAxNjAuMDcgMTI0LjUxMi0xMDQuNDc4eiIvPjxwYXRoIGQ9Ik0xLjA1NCA4OTIuNzdsMTcuMDc4IDk2Ljg1OCA3NS4zNDItNjMuMjJ6Ii8+PC9nPjwvZz48L3N2Zz4=);
  background-size: 100% 100%;
  background-repeat: no-repeat;
  position: relative;
}

.login-broadside .describe {
  position: absolute;
  bottom: 50px;
  left: 0;
  right: 0;
  width: 80%;
  margin: 0 auto;
  color: rgba(230, 230, 230, 0.6);
  line-height: 23px;
  font-size: 14px;
}

@media screen and (max-height: 500px) {
  .copyright {
    display: none;
  }
}

@media screen and (max-width: 1000px) {
  .login-broadside {
    display: none;
  }
}

</style>
