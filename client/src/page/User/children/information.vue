<template>
  <div>
    <ctm-shelf title="账户资料">
      <div slot="content">
        <div class="avatar-box">
          <div class=img-box><img :src="userInfo.info.avatar" alt=""></div>
          <div class="r-box">
            <h3>修改头像</h3>
            <ctm-button text="更换头像" classStyle="main-btn" style="margin: 0;" @btnClick="editAvatar()"></ctm-button>
          </div>
        </div>
        <div class="edit-avatar" v-if="editAvatarShow">
          <ctm-shelf title="设置头像">
        <span slot="right">
                              <span class="close" @click="editAvatarShow=false">
                        <svg t="1501234940517" class="icon" style="" viewBox="0 0 1024 1024" version="1.1"
                             xmlns="http://www.w3.org/2000/svg" p-id="3014" xmlns:xlink="http://www.w3.org/1999/xlink"
                             width="20" height="20"><path
                          d="M941.576 184.248l-101.824-101.824L512 410.176 184.248 82.424 82.424 184.248 410.168 512l-327.744 327.752 101.824 101.824L512 613.824l327.752 327.752 101.824-101.824L613.832 512z"
                          fill="#cdcdcd" p-id="3015"></path></svg>
                    </span>
        </span>
            <div slot="content" class="content">
              <div class="edit-l">
                <div style="width: 100px;height: 100px;border: 1px solid #ccc;margin-bottom: 20px;overflow: hidden;">
                  <div class="show-preview"
                       :style="{'width': previews.w + 'px','height': previews.h + 'px','overflow': 'hidden','zoom':option.zoom}">
                    <div :style="previews.div">
                      <img :src="option.img" :style="previews.img">
                    </div>
                  </div>
                </div>
                <div style="padding: 10px 0">头像预览</div>
                <div class="btn">
                  <a href="javascript:;">重新选择</a>
                  <input type="file" accept="image/*" value="上传头像" @change="upimg($event)"></div>
              </div>
              <div class="edit-r">
                <div>
                  <div class="big" id="cropper-target" v-if="option.img||userInfo.info.avatar">
                    <vueCropper
                      :img="option.img||userInfo.info.avatar"
                      @realTime="realTime"
                      ref="cropper"
                      :outputSize="example2.size"
                      :info="example2.info"
                      :canMove="example2.canMove"
                      :canScale="example2.canScale"
                      :autoCrop="example2.autoCrop"
                      :autoCropWidth="example2.width"
                      :autoCropHeight="example2.height"
                      :fixed="example2.fixed"
                    ></vueCropper>
                  </div>
                </div>

              </div>
              <div class="bootom-btn pa">
                <ctm-button style="width: 140px;height: 40px;line-height: 40px"
                          text="取消"
                          @btnClick="editAvatarShow=false">
                </ctm-button>
                <ctm-button style="width: 140px;height: 40px;line-height: 40px"
                          text="确定"
                          classStyle="main-btn"
                          @btnClick="cropper">
                </ctm-button>
              </div>
            </div>
          </ctm-shelf>
        </div>
      </div>
    </ctm-shelf>
  </div>
</template>
<script>
  import CtmButton from '/components/ctmButton'
  import { upload, updateAvatar } from '/api/index'
  import CtmShelf from '/components/shelf'
  import vueCropper from 'vue-cropper'
  import { mapState, mapMutations } from 'vuex'
  export default {
    data () {
      return {
        editAvatarShow: false,
        cropContext: '',
        cropperImg: '',
        previews: {},
        option: {
          img: '',
          zoom: 0
        },
        fixedNumber: [1, 1],
        example2: {
          info: false,
          size: 1,
          canMove: false,
          canScale: true,
          autoCrop: true,
          // 只有自动截图开启 宽度高度才生效
          autoCropWidth: 100,
          autoCropHeight: 100,
          // 开启宽度和高度比例
          fixed: true
        }
      }
    },
    computed: {
      ...mapState(['userInfo'])
    },
    mounted () {
      this.option.img = this.userInfo.info.avatar
    },
    methods: {
      ...mapMutations([
        'RECORD_USERINFO'
      ]),
      upimg (e) {
        var file = e.target.files[0]
        if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
          this.$message({
            type: 'error',
            message: '图片类型必须是.gif,jpeg,jpg,png,bmp中的一种!',
            customClass: 'tips-msg'
          })
          return false
        }
        if (file.size > 2 * 1024 * 1024) {
          this.$message({
            type: 'error',
            message: '选择的图片不能大于2M！',
            customClass: 'tips-msg',
            showClose: true
          })
          return false
        }
        var reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = (e) => {
          this.option.img = e.target.result
        }
      },
      cropper () {
        if (this.option.img) {
          this.$refs.cropper.getCropData((data) => {
            upload({imgData: data}).then(res => {
              if (res.status === '0') {
                let path = res.result.path
                updateAvatar({imageSrc: path}).then(res1 => {
                  if (res1.status === '0') {
                    let info = this.userInfo.info
                    info.avatar = path
                    this.RECORD_USERINFO({info})
                    this.$message({
                      type: 'success',
                      message: '更换头像成功！',
                      customClass: 'tips-msg'
                    })
                    this.editAvatarShow = false
                  } else {
                    this.$message({
                      type: 'error',
                      message: '更换失败！请稍后重试！',
                      customClass: 'tips-msg'
                    })
                  }
                })
              }
            })
          })
        } else {
          this.$message({
            type: 'warning',
            message: '未发现图片！请点击左侧按钮重新上传！',
            customClass: 'tips-msg'
          })
        }
      },
      editAvatar () {
        this.editAvatarShow = true
      },
      realTime (data) {
        this.previews = data
        let w = 100 / data.w
        this.option.zoom = w
      }
    },
    components: {
      CtmButton,
      CtmShelf,
      vueCropper
    }
  }
</script>
<style lang="scss" scoped>
  @import "../../../assets/style/mixin";

  .avatar-box {
    height: 124px;
    display: flex;
    margin: 0 30px 30px;
    border-bottom: #dadada solid 1px;
    line-height: 30px;
    display: flex;
    align-items: center;
    .img-box {
      @include wh(80px);
      border-radius: 5px;
      overflow: hidden;
    }
    img {
      display: block;
      @include wh(100%)
    }
    .r-box {
      margin-left: 20px;
      h3 {
        font-size: 18px;
        font-weight: 400;
        color: #333;
      }
    }
  }

  // 修改头像
  .edit-avatar {
    z-index: 999;
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    @include wh(100%);
    background-color: rgba(0, 0, 0, .5);
    @extend %block-center;
    .content {
      display: flex;
      padding: 45px 100px 0;
    }
    > div {
      box-sizing: content-box;
      @include wh(700px, 500px);
      margin: 0;
    }
    .btn {
      width: 80px;
      height: 30px;
      margin: 10px;
      position: relative;
      text-align: center;
      line-height: 30px;
      text-shadow: rgba(255, 255, 255, .496094) 0 1px 0;
      border: 1px solid #dadada;
      border-radius: 6px;
      box-sizing: content-box;
      a {
        color: #888;
        font-size: 12px;
        display: block;
        @include wh(100%);
      }
    }
    input[type=file] {
      position: absolute;
      right: 0;
      left: 0;
      top: 0;
      opacity: 0;
      width: 80px;
      height: 30px;
      cursor: pointer;
      box-sizing: border-box;
      border: 15px solid #000;
      overflow: hidden;
    }
    .edit-l {
      width: 100px;
      text-align: center;
    }
    .edit-r {
      margin-left: 100px;
      flex: 1;
      > div {
        border: 1px solid #ccc;
        width: 250px;
        height: 250px;
      }
    }
  }

  .image-container {
    width: 100px;
    height: 100px;
    border: 1px solid #ccc;
  }

  .close {
    position: absolute;
    right: 10px;
    top: 0;
    bottom: 0;
    padding: 0 10px;
    @extend %block-center;
    &:hover {
      svg {
        transition: all 1s;
        transform: rotate(360deg);
        transform-origin: 50% 50%;
      }
      path {
        fill: #8a8a8a;
      }
    }
  }

  .big {
    display: block;
    width: 250px;
    height: 250px;
  }

  .bootom-btn {
    padding: 0 15px;
    border-top: 1px solid #E6E6E6;
    bottom: 0;
    height: 60px;
    right: 0;
    left: 0;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .tips-msg {
    position: relative;
    z-index: 9999;
  }
</style>
