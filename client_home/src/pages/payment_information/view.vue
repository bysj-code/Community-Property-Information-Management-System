<template>
  <view>
    <view class="container diy_view">
      <view>
        <view>
          <view class="">
            <uni-forms :modelValue="form"  v-if="is_view()">

              <uni-forms-item v-if="$check_field('get','owner') || $check_field('add','owner') || $check_field('set','owner')" label="业主" name="owner">
                    <uni-data-select
                  id="form_owner"
                  v-model="form['owner']"
                  :localdata="list_user_owner"
                  :clear="!disabledObj['owner_isDisabled']"
                  :disabled="disabledObj['owner_isDisabled']"
                  v-if="user_group === '管理员' || (form['payment_information_id'] && $check_field('set','owner')) || (!form['payment_information_id'] && $check_field('add','owner'))"
                ></uni-data-select>
                <uni-data-select
                  v-model="form['owner']"
                  :localdata="list_user_owner"
                  :clear="false"
                  :disabled="true"
                  v-else-if="$check_field('get','owner')" id="owner"
                ></uni-data-select>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','month') || $check_field('add','month') || $check_field('set','month')" label="月份" name="month">
                            <uni-easyinput type="text" v-model="form['month']" v-if="user_group === '管理员' || (form['payment_information_id'] && $check_field('set','month')) || (!form['payment_information_id'] && $check_field('add','month'))" :disabled="disabledObj['month_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','month')">
                  {{ form['month'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','room_number') || $check_field('add','room_number') || $check_field('set','room_number')" label="房号" name="room_number">
                            <uni-easyinput type="text" v-model="form['room_number']" v-if="user_group === '管理员' || (form['payment_information_id'] && $check_field('set','room_number')) || (!form['payment_information_id'] && $check_field('add','room_number'))" :disabled="disabledObj['room_number_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','room_number')">
                  {{ form['room_number'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','payment_name') || $check_field('add','payment_name') || $check_field('set','payment_name')" label="缴费名称" name="payment_name">
                            <uni-easyinput type="text" v-model="form['payment_name']" v-if="user_group === '管理员' || (form['payment_information_id'] && $check_field('set','payment_name')) || (!form['payment_information_id'] && $check_field('add','payment_name'))" :disabled="disabledObj['payment_name_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','payment_name')">
                  {{ form['payment_name'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','payment_amount') || $check_field('add','payment_amount') || $check_field('set','payment_amount')" label="缴费金额" name="payment_amount">
                            <uni-easyinput type="text" v-model="form['payment_amount']" v-if="user_group === '管理员' || (form['payment_information_id'] && $check_field('set','payment_amount')) || (!form['payment_information_id'] && $check_field('add','payment_amount'))" :disabled="disabledObj['payment_amount_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','payment_amount')">
                  {{ form['payment_amount'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','remarks') || $check_field('add','remarks') || $check_field('set','remarks')" label="备注" name="remarks">
                    <uni-easyinput type="textarea" v-model="form['remarks']" v-if="user_group === '管理员' || (form['payment_information_id'] && $check_field('set','remarks')) || (!form['payment_information_id'] && $check_field('add','remarks'))" :disabled="disabledObj['remarks_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','remarks')">
                  {{ form['remarks'] }}
                </text>
                  </uni-forms-item>


            </uni-forms>
            <view class="form_button">
              <button size="mini" type="primary" @click="submit()" class="primary_btn">提交</button>
              <button size="mini" @click="cancel()">取消</button>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import mixin from "@/libs/mixins/page.js";

export default {
  mixins: [mixin],
  data() {
    return {
      field: "payment_information_id",
      url_add: "~/api/payment_information/add?",
      url_set: "~/api/payment_information/set?",
      url_get_obj: "~/api/payment_information/get_obj?",
      url_upload: "~/api/payment_information/upload?",

      query: {
        "payment_information_id": 0,
      },

      form: {
            "owner": 0, // 业主
                    "month":  '', // 月份
                    "room_number":  '', // 房号
                    "payment_name":  '', // 缴费名称
                    "payment_amount":  0 , // 缴费金额
                    "remarks":  '', // 备注
                                    "payment_information_id": 0, // ID
                
              },
          disabledObj:{
                        "owner_isDisabled": false,
                                "month_isDisabled": false,
                                "room_number_isDisabled": false,
                                "payment_name_isDisabled": false,
                                                    "remarks_isDisabled": false,
                                  },
                                // 用户列表
            list_user_owner: [],
                                                                                                                                              }
  },
  methods: {
    changeLog(v,value){
      this.form[value] = v
    },
    /**
     * 上传文件
     * @param {Object} param文件参数
     */
    change_file(key_name){
      var _self = this;
      // 选择图像方法
      uni.chooseFile({
        count: 1,
        sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
        sourceType: ['album'], //从相册选择
        success: function(res) {
          const tempFilePaths = res.tempFilePaths;
          const uploadTask = uni.uploadFile({
            url: _self.$fullUrl('/api/payment_information/upload?'),
            filePath: tempFilePaths[0],
            name: 'file',
            formData: {
              'i_want_to_customize': 'test'
            },
            header: {
              'x-auth-token': _self.$store.state.user.token
            },
            success: function(uploadFileRes) {
              var filename = JSON.parse(uploadFileRes.data).result.url
              _self.form[key_name] = filename
            }
          });

          uploadTask.onProgressUpdate(function(res) {
            _self.percent = res.progress;
            console.log('上传进度' + res.progress);
            console.log('已经上传的数据长度' + res.totalBytesSent);
            console.log('预期需要上传的数据总长度' + res.totalBytesExpectedToSend);
          });
        },
        error: function(e) {
          console.log(e);
        }
      });
    },
    /**
     * 上传图片
     * @param {Object} param文件参数
     */
    change_img(key_name){
      var _self = this;
      _self.upload_img_flag = false
      // 选择图像方法
      uni.chooseImage({
        count: 1,
        sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
        sourceType: ['album'], //从相册选择
        success: function(res) {
          const tempFilePaths = res.tempFilePaths;
          const uploadTask = uni.uploadFile({
            url: _self.$fullUrl('/api/payment_information/upload?'),
            filePath: tempFilePaths[0],
            name: 'file',
            formData: {
              'i_want_to_customize': 'test'
            },
            header: {
              'x-auth-token': _self.$store.state.user.token
            },
            success: function(uploadFileRes) {
              var filename = JSON.parse(uploadFileRes.data).result.url
              _self.form[key_name] = filename
            }
          });

          uploadTask.onProgressUpdate(function(res) {
            _self.percent = res.progress;
            console.log('上传进度' + res.progress);
            console.log('已经上传的数据长度' + res.totalBytesSent);
            console.log('预期需要上传的数据总长度' + res.totalBytesExpectedToSend);
          });
        },
        error: function(e) {
          console.log(e);
        }
      });
    },
    /**
     * 获取对象后获取缓存表单
     * @param {Object} json
     * @param {Object} func
     */
    get_obj_before(param){
      var form = uni.db.get("form");
      if (form) {
        delete(form.examine_state)
        delete(form.examine_reply)
        this.obj = uni.push(this.obj ,form);
        this.form = uni.push(this.form ,form);
      }
      var arr = []
      for (let key in form) {
        arr.push(key)
      }
      for (var i=0;i<arr.length;i++){
        this.disabledObj[arr[i] + '_isDisabled'] = true
      }
                                                      uni.db.del("form");
      return param;
    },
            /**
     * 获取居民用户用户列表
     */
    async get_list_user_owner() {
      // if(this.user_group !== "管理员" && this.form["owner"] === 0) {
      //     this.form["owner"] = this.user.user_id;
      // }
      var json = await this.$get("~/api/user/get_list?user_group=居民用户");
      if(json.result && json.result.list){
        json.result.list.map((o) => this.list_user_owner.push({value:o.user_id,text:o.nickname + '-' + o.username}));
      }
      else if(json.error){
        console.error(json.error);
      }
    },
            
            
            
            
            
            
    
    /**
     * 获取对象之后
     * @param {Object} json
     * @param {Object} func
     */
    get_obj_after(json, func){
                                                    },

    is_view(){
      var bl = this.user_group == "管理员";

      if(!bl){
        bl = this.$check_action('/payment_information/table','add');
        console.log(bl ? "你有表格添加权限视作有添加权限" : "你没有表格添加权限");
      }
      if(!bl){
        bl = this.$check_action('/payment_information/table','set');
        console.log(bl ? "你有表格添加权限视作有修改权限" : "你没有表格修改权限");
      }
      if(!bl){
        bl = this.$check_action('/payment_information/view','add');
        console.log(bl ? "你有视图添加权限视作有添加权限" : "你没有视图添加权限");
      }
      if(!bl){
        bl = this.$check_action('/payment_information/view','set');
        console.log(bl ? "你有视图修改权限视作有修改权限" : "你没有视图修改权限");
      }
      if(!bl){
        bl = this.$check_action('/payment_information/view','get');
        console.log(bl ? "你有视图查询权限视作有查询权限" : "你没有视图查询权限");
      }

      console.log(bl ? "具有当前页面的查看权，请注意这不代表你有字段的查看权" : "无权查看当前页，请注意即便有字段查询权限没有页面查询权限也不行");

      return bl;
    },

  },
  created() {
            this.get_list_user_owner();
                                                      },
}
</script>

<style scoped>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}

.form_button{
  padding-bottom: 15px;
  display: flex;
}
.form_button button{
  width: 40%;
}
.query_select{
  border-color: rgb(229, 229, 229);
  background-color: rgb(255, 255, 255);
  border-radius: 4px;
  box-sizing: border-box;
  flex: 1;
  width: 100%;
  line-height: 2;
  font-size: 14px;
  height: 2.4em;
  min-height: 2.4em;
  display: block;
  outline:none;
}

.query_option{
  width: 100%;
}

.btn_add_img{
  color: #D3D3D3;
  text-align: center;
  border: 1px solid #eee;
  height: 5rem;
  width: 5rem;
  position: relative;
}
.btn_add_img text{
  font-size: 35px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}

.form_button {
  padding-bottom: 15px;
  display: flex;
}
.form_button button {
  width: 40%;
}
.query_select {
  border-color: rgb(229, 229, 229);
  background-color: rgb(255, 255, 255);
  border-radius: 4px;
  box-sizing: border-box;
  flex: 1;
  width: 100%;
  line-height: 2;
  font-size: 14px;
  height: 2.4em;
  min-height: 2.4em;
  display: block;
  outline: none;
}

.query_option {
  width: 100%;
}

.btn_add_img {
  color: #d3d3d3;
  text-align: center;
  border: 1px solid #eee;
  height: 5rem;
  width: 5rem;
  position: relative;
}
.btn_add_img text {
  font-size: 35px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
/*新样式*/
.uni-forms{
padding-top:1rem;
}
.uni-forms-item {
	padding: 6px 10px;
    background: #f8f6fc;
}
.uni-forms .is-input-border{
	border: 0;
}
.container{
	    -webkit-box-shadow: 0px 0px 0px #888888;
	    box-shadow: 0px 0px 0px #888888;
}
.form_button .primary_btn{
		background-color: #22B8B8;
		color: #FFFFFF;
	}
</style>
