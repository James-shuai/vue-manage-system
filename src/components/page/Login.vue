<template>
    <div class="login-wrap">
        <div class="ms-login">
            <div class="ms-title">后台管理系统</div>
            <el-form :model="param"  ref="login" label-width="0px" class="ms-content">
                <el-form-item prop="username">
                    <el-input v-model="param.username" placeholder="username">
                        <el-button slot="prepend" icon="el-icon-lx-people"></el-button>
                    </el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input
                        type="password"
                        placeholder="password"
                        v-model="param.password"
                        @keyup.enter.native="submitForm()"
                    >
                        <el-button slot="prepend" icon="el-icon-lx-lock"></el-button>
                    </el-input>
                </el-form-item>
                <div class="login-btn">
                    <el-button type="primary" @click="submitForm()">登录</el-button>
                </div>
                <p class="login-tips">Tips : 用户名test，密码1234。</p>
            </el-form>
        </div>
    </div>
</template>

<script>
let Base64 = require('js-base64').Base64;
export default {
    data: function() {
        return {
            param: {
                username: 'test',
                password: '1234',
            },
            menuList:'',

        };
    },
    methods: {
        submitForm() {
            this.$refs.login.validate(valid => {
                if (valid) {
                  //用户名密码校验
                  var encode = Base64.encode('cloud-pc:cloud-secret');
                  var postdatsss =
                    "grant_type=" +
                    "password&username=" +
                    this.param.username +
                    "&password=" +
                    this.param.password;
                    const loading = this.$loading({
                        lock: true,
                        text: '登陆中',
                        spinner: 'el-icon-loading',
                        background: 'rgba(0, 0, 0, 0.7)'
                    });
                  this.$request({
                    url:'/auth/oauth/token',
                    method:'post',
                    headers:{
                      'Content-Type': 'application/x-www-form-urlencoded',
                      'Authorization': 'Basic '+encode
                    },
                    data:postdatsss
                  }).then(res =>{
                    localStorage.setItem('token', res.access_token);
                      //获取用户权限
                      var token =
                          "token=" +
                          res.access_token;
                      this.$request({
                          url:'/auth/oauth/check_token',
                          method:'post',
                          headers:{
                              'Content-Type': 'application/x-www-form-urlencoded',
                              'Authorization': 'Basic '+encode
                          },
                          data:token
                      }).then(response=>{
                          if (response.authorities !=null) {
                              localStorage.setItem('authorities',response.authorities)
                          }else {
                              this.$message.error('登陆异常');
                          }
                      })
                    //获取菜单
                    this.$request({
                      url:'/system/menu/getNavMenu',
                      method:'post',
                      headers:{
                        'Authorization': 'Bearer '+res.access_token
                      },
                    }).then(result=>{
                      if (result.code === 200) {
                          loading.close()
                          this.$message.success('登录成功');
                          localStorage.setItem('menuList',JSON.stringify(result.data))
                          localStorage.setItem('ms_username', result.message);
                        this.menuList=result.data;
                        this.$router.push('/');
                      }
                    })
                  }).catch(error => {
                    this.$message.error('用户名或密码错误');
                    console.log('异常：'+error)
                      loading.close()
                  })

                } else {
                    this.$message.error('请输入账号和密码');
                    console.log('error submit!!');
                    return false;
                }
            });
        },
    },
};
</script>

<style scoped>
.login-wrap {
    position: relative;
    width: 100%;
    height: 100%;
    background-image: url(../../assets/img/login-bg.jpg);
    background-size: 100%;
}
.ms-title {
    width: 100%;
    line-height: 50px;
    text-align: center;
    font-size: 20px;
    color: #fff;
    border-bottom: 1px solid #ddd;
}
.ms-login {
    position: absolute;
    left: 50%;
    top: 50%;
    width: 350px;
    margin: -190px 0 0 -175px;
    border-radius: 5px;
    background: rgba(255, 255, 255, 0.3);
    overflow: hidden;
}
.ms-content {
    padding: 30px 30px;
}
.login-btn {
    text-align: center;
}
.login-btn button {
    width: 100%;
    height: 36px;
    margin-bottom: 10px;
}
.login-tips {
    font-size: 12px;
    line-height: 30px;
    color: #fff;
}
</style>
