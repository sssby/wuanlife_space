<template>
  <div id="personal-data" class="personal-data view-container wl-card">
      <section>
          <h1>个人资料</h1>
      <div class="personal-data-form" v-loading="loading1">
          <div class="form-left" v-loading="loading">
              <img v-bind:src="dafaultAvatarUrl" id="avatar" ref="avatar">
              <el-upload
                :action="UPLOAD_ADDRESS"
                :before-upload='beforeUpload'
                :data="uploadData"
                :on-success='upScuccess'
                :on-error="upError"
                ref="upload"
                style="display:none">
                <el-button id="img-input"
                 size="small"
                 type="primary">点击上传</el-button>
                </el-upload>
              <button @click="changeAvatar"><icon-svg icon-class="modify" class="avatar-icon"></icon-svg>修改</button>
          </div>
          <div class="form-right">
              <div class="form-item">
                  <span>邮箱</span>
                  <p>{{ mail }}</p>
              </div>
              <div class="form-item">
                  <span>昵称</span>
                  <input type="text" v-model="name">
              </div>
              <div class="form-item">
                  <span>性别</span>
                  <div class="form-item-sex">
                    <input type="radio" id="male" value="male" v-model="sex">
                    <label for="male" :class="[sex === 'male' ? 'label-active' : '']">男</label>
                    <input type="radio" id="female" value="female" v-model="sex">
                    <label for="female" :class="[sex === 'female' ? 'label-active' : '']">女</label>
                    <input type="radio" id="secrecy" value="secrecy" v-model="sex">
                    <label for="secrecy" :class="[sex === 'secrecy' ? 'label-active' : '']">不想透露</label>
                  </div>
              </div>
              <div class="form-item">
                  <span>生日</span>
                  <div class="form-item-date">
                      <date-picker
                        :min="1970"
                        :max="2018"
                        :defaultNum="yearNumber"
                        v-on:pick="year"
                        class="date-picker"></date-picker>年
                      <date-picker
                        :min="1"
                        :max="12"
                        :defaultNum="mouthNumber"
                        v-on:pick="mouth"
                        class="date-picker"></date-picker>月
                      <date-picker
                        :min="1"
                        :max="dayMax"
                        :defaultNum="dayNumber"
                        v-on:pick="day"
                        class="date-picker"></date-picker>日
                  </div>
              </div>
          </div>
      </div>
      <el-button class="wl-btn save" @click="pushPersonalData" :loading="loading2">保存</el-button>
      </section>
  </div>
</template>

<script>
import DatePicker from 'components/DatePicker'
import { getToken } from 'api/qiniu'
import { getUser } from 'api/user'
import { mapGetters } from 'vuex'
import { Notification } from 'element-ui'

const QINIU_DOMAIN = '//7xlx4u.com1.z0.glb.clouddn.com/' // 图片服务器域名，展示时用

export default {
  name: 'personalData',
  components: {
    DatePicker
  },
  data () {
    return {
      yearNumber: 1970,
      mouthNumber: 1,
      dayNumber: 1,
      dayMax: 31,
      leap: false,
      sex: '',
      mail: '',
      name: '',
      dafaultAvatarUrl: 'http://7xlx4u.com1.z0.glb.clouddn.com/o_1aqt96pink2kvkhj13111r15tr7.jpg?imageView2/1/w/97/h/97',
      UPLOAD_ADDRESS: location.protocol === 'http:' ? 'http://upload.qiniu.com' : 'https://up.qbox.me',
      uploadData: {},
      loading: false,
      loading1: false,
      loading2: false,
      default: {}
    }
  },
  computed: {
    birthday: {
      get: function () {
        let day = this.dayNumber < 10 ? '0' + this.dayNumber : this.dayNumber
        let mouth = this.mouthNumber < 10 ? '0' + this.mouthNumber : this.mouthNumber
        return `${this.yearNumber}-${mouth}-${day}`
        // return new Date(Date.UTC(this.yearNumber, this.mouthNumber - 1, this.dayNumber))
      },
      set: function (val) {
        let time = new Date(val)
        this.year(time.getFullYear())
        this.mouth(time.getMonth() + 1)
        this.day(time.getDate())
      }
    },
    ...mapGetters(['user'])
  },
  methods: {
    year: function (val) {
      this.yearNumber = val

      // 判断闰年
      let isLeap = (this.yearNumber % 4 === 0 && this.yearNumber % 100 !== 0) || this.yearNumber % 400 === 0
      if (isLeap) {
        this.leap = true
      } else {
        this.leap = false
      }
    },
    mouth: function (val) {
      this.mouthNumber = val

      // 相应改变每月的天数
      if (this.mouthNumber === 1 || this.mouthNumber === 3 || this.mouthNumber === 5 || this.mouthNumber === 7 || this.mouthNumber === 8 || this.mouthNumber === 10 || this.mouthNumber === 12) {
        this.dayMax = 31
      } else {
        this.dayMax = 30
      }
      if (this.mouthNumber === 2) {
        if (this.leap === true) {
          this.dayMax = 29
        } else {
          this.dayMax = 28
        }
      }
    },
    day: function (val) {
      this.dayNumber = val
    },
    changeAvatar: function () {
      const self = this
      if (this.loading) {
        return
      }
      this.loading = true
      setTimeout(function () {
        self.loading = false
      }, 10000)
      document.getElementById('img-input').click()
    },
    async pushPersonalData () {
      if (this.loading2) {
        return
      }
      this.loading2 = true
      var changeUser = {}
      if (this.default.name !== this.name) {
        changeUser.name = this.name
        this.default.name = this.name
      }
      if (this.default.sex !== this.sex) {
        changeUser.sex = this.sex
        this.default.sex = this.sex
      }
      if (this.default.birthday !== this.birthday) {
        changeUser.birthday = this.birthday
        this.default.birthday = this.birthday
      }
      if (this.default.avatar_url !== this.$refs.avatar.getAttribute('src')) {
        changeUser.avatar_url = this.$refs.avatar.getAttribute('src')
        this.default.avatar_url = this.$refs.avatar.getAttribute('src')
      }
      if (changeUser.name === undefined && changeUser.sex === undefined && changeUser.birthday === undefined && changeUser.avatar_url === undefined) {
        this.loading2 = false
        Notification.warning({
          message: '请改变个人资料中某一项后，再提交！',
          offset: 60
        })
        return
      }
      try {
        this.$store.dispatch('PutUser', changeUser).then(res => {
          Notification.success({
            message: '修改个人资料成功！',
            offset: 60
          })
        })
      } catch (e) {
        if (e.data) {
          Notification.error({
            message: e.data.error,
            offset: 60
          })
        } else {
          console.log(e)
        }
      }
      this.loading2 = false
    },
    beforeUpload: function (file) {
      return this.qnUpload(file)
    },
    qnUpload: function (file) {
      const suffix = file.name.split('.')
      const ext = suffix.splice(suffix.length - 1, 1)[0]
      // TODO: 图片格式/大小限制
      return getToken().then(res => {
        this.uploadData = {
          key: `image/${suffix.join('.')}_${new Date().getTime()}.${ext}`,
          token: res.uploadToken
        }
      })
    },
    upScuccess: function (e, file, fileList) {
      const url = QINIU_DOMAIN + e.key
      this.$refs.avatar.setAttribute('src', url)
      this.loading = false
    },
    upError: function (e, file, fileList) {
      this.loading = false
    },
    async getUserData () {
      try {
        await getUser().then(res => {
          this.mail = res.mail
          this.sex = res.sex
          this.name = res.name
          this.birthday = res.birthday
          this.default = res
          let isDefault = res.avatar_url === 'default_url'
          if (!isDefault) {
            this.dafaultAvatarUrl = res.avatar_url
          }
        })
      } catch (e) {
        if (e.data) {
          Notification.error({
            message: e.data.error,
            offset: 60
          })
        } else {
          console.log(e)
        }
      }
      this.loading1 = false
    }
  },
  mounted () {
    this.loading1 = true
    this.getUserData()
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
#personal-data{
    width: 720px;
    margin-top: 119px;
    margin-bottom: 36px;
    background-color: #fff;
    border-radius: 4px;
    padding: 42px 111px 68px 121px;
    text-align: center;
    section{
        h1{
            font-size: 20px;
            color: #5677fc;
            margin-bottom: 42px;
        }
        .personal-data-form{
            display: flex;
            justify-content: space-between;
            .form-left{
                img{
                    display: block;
                    height: 97px;
                    width: 97px;
                    border-radius: 100%;
                    background-color: rgb(165, 164, 164);
                    box-shadow: 0px 4px 5px 0px rgba(181, 181, 181, 0.75);
                    margin-bottom: 7px;
                }
                button{
                    border: 0;
                    font-size: 11px;
                    color: #5677fc;
                    background-color: transparent;
                    cursor: pointer;
                    .avatar-icon{
                        margin-right: 6 px;
                    }
                }
            }
            .form-right{
                border-left: solid 2px #c9c9c9;
                padding-left: 26px;
                .form-item{
                    display: flex;
                    min-height: 45px;
                    align-items: center;
                    font-size: 17px;
                    color: #434343;
                    margin-bottom: 37px;
                    &:last-child{
                        margin-bottom: 0;
                    }
                    span{
                        margin-right: 32px;
                        font-size: 15px;
                        color: #434343;
                    }
                    &>input{
                        height: 45px;
                        width: 253px;
                        color: #434343;
                        box-shadow: 0px 3px 7px 0px
                    rgba(99, 99, 99, 0.16);
                      border-radius: 4px;
                      border: solid 2px rgba(171, 171, 171, 0.45);
                        padding: 0 16px;
                    }
                    &>input:focus{
                        background-color: rgba(248, 249, 250, 0.4);
                      box-shadow: 0px 3px 7px 0px
                    rgba(86, 119, 252, 0.16);
                      border-radius: 4px;
                      border: solid 2px rgba(0, 64, 185, 0.4);
                    }
                    .form-item-sex{
                        display: flex;
                        align-items: center;
                        justify-content: flex-start;
                        width: 253px;
                        input{
                            width: 0;
                            height: 0;
                        }
                        label{
                            width: 65px;
                            height: 44px;
                            line-height: 44px;
                            display: inline-block;
                            cursor: pointer;
                            background-color: rgba(248, 249, 250, 0.45);
                          box-shadow: 0px 3px 7px 0px
                        rgba(99, 99, 99, 0.16);
                          border-radius: 4px;
                          border: solid 2px rgba(171, 171, 171, 0.45);
                            font-size: 17px;
                            color: #434343;
                            font-weight: normal;
                            margin-right: 29px;
                            &:last-child{
                                font-size: 13px;
                                margin-right: 0;
                            }
                        }
                        .label-active{
                            background-color: rgba(248, 249, 250, 0.4);
                          box-shadow: 0px 3px 7px 0px
                        rgba(86, 119, 252, 0.16);
                          border: solid 2px rgba(0, 64, 185, 0.4);
                        }
                    }
                    .form-item-date{
                        display: flex;
                        justify-content: space-between;
                        align-items: center;
                        .date-picker{
                            margin-left: 8px;
                            z-index: 0;
                            &:first-child{
                                margin-left: 0;
                            }
                        }
                    }
                }
            }
        }
        .save{
            width: 150px;
            margin-top: 66px;
        }
    }
}
</style>
