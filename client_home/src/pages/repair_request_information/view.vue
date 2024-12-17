<template>
  <view>
    <view class="container diy_view">
      <view>
        <view>
          <view class="">
            <uni-forms :modelValue="form"  v-if="is_view()">

              <uni-forms-item v-if="$check_field('get','owner_user') || $check_field('add','owner_user') || $check_field('set','owner_user')" label="业主用户" name="owner_user">
                    <uni-data-select
                  id="form_owner_user"
                  v-model="form['owner_user']"
                  :localdata="list_user_owner_user"
                  :clear="!disabledObj['owner_user_isDisabled']"
                  :disabled="disabledObj['owner_user_isDisabled']"
                  v-if="user_group === '管理员' || (form['repair_request_information_id'] && $check_field('set','owner_user')) || (!form['repair_request_information_id'] && $check_field('add','owner_user'))"
                ></uni-data-select>
                <uni-data-select
                  v-model="form['owner_user']"
                  :localdata="list_user_owner_user"
                  :clear="false"
                  :disabled="true"
                  v-else-if="$check_field('get','owner_user')" id="owner_user"
                ></uni-data-select>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','problems_reported_for_repair') || $check_field('add','problems_reported_for_repair') || $check_field('set','problems_reported_for_repair')" label="报修问题" name="problems_reported_for_repair">
                            <uni-easyinput type="text" v-model="form['problems_reported_for_repair']" v-if="user_group === '管理员' || (form['repair_request_information_id'] && $check_field('set','problems_reported_for_repair')) || (!form['repair_request_information_id'] && $check_field('add','problems_reported_for_repair'))" :disabled="disabledObj['problems_reported_for_repair_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','problems_reported_for_repair')">
                  {{ form['problems_reported_for_repair'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','site_photos') || $check_field('add','site_photos') || $check_field('set','site_photos')" label="现场照片" name="site_photos">
                    <!-- 修改权限 -->
                <view class="diy_field diy_img" v-if="form['site_photos'] && $check_field('set','site_photos')">
                  <image v-if="disabledObj['site_photos_isDisabled']" :src="$fullUrl(form['site_photos'])" />
                  <image v-if="!disabledObj['site_photos_isDisabled']" :src="$fullUrl(form['site_photos'])" @click="change_img('site_photos')" />
                </view>
                <!-- 添加权限 -->
                <view class="diy_field diy_img" v-else-if="!form['site_photos'] && $check_field('add','site_photos')">
                  <view v-if="disabledObj['site_photos_isDisabled']" class="btn_add_img">
                    <text>+</text>
                  </view>
                  <view v-if="!disabledObj['site_photos_isDisabled']" class="btn_add_img" @click="change_img('site_photos')">
                    <text>+</text>
                  </view>
                </view>
                <!-- 查询权限 -->
                <view class="diy_field diy_img" v-else-if="$check_field('get','site_photos')">
                  <image :src="$fullUrl(form['site_photos'])" />
                </view>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','repair_location') || $check_field('add','repair_location') || $check_field('set','repair_location')" label="报修地点" name="repair_location">
                            <uni-easyinput type="text" v-model="form['repair_location']" v-if="user_group === '管理员' || (form['repair_request_information_id'] && $check_field('set','repair_location')) || (!form['repair_request_information_id'] && $check_field('add','repair_location'))" :disabled="disabledObj['repair_location_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','repair_location')">
                  {{ form['repair_location'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','problem_description') || $check_field('add','problem_description') || $check_field('set','problem_description')" label="问题描述" name="problem_description">
                    <uni-easyinput type="textarea" v-model="form['problem_description']" v-if="user_group === '管理员' || (form['repair_request_information_id'] && $check_field('set','problem_description')) || (!form['repair_request_information_id'] && $check_field('add','problem_description'))" :disabled="disabledObj['problem_description_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','problem_description')">
                  {{ form['problem_description'] }}
                </text>
                  </uni-forms-item>
              <uni-forms-item label="审核状态" name="examine_state">
                <uni-data-select
                    v-model="form['examine_state']"
                    :localdata="list_examine_state"
                    v-if="user_group === '管理员' || (form['examine_state'] && $check_examine()) || (!form['examine_state'] && $check_examine())"
                ></uni-data-select>
                <text v-else>{{form["examine_state"]}}</text>
              </uni-forms-item>
              <uni-forms-item label="审核回复" name="examine_reply">
                <uni-easyinput type="text" placeholder="请输入审核回复" v-model="form['examine_reply']"
                               v-if="user_group === '管理员' || (form['examine_reply'] && $check_examine()) || (!form['examine_reply'] && $check_examine())" />
                <!-- 仅查看 -->
                <text v-else>{{form["examine_reply"]}}</text>
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
      field: "repair_request_information_id",
      url_add: "~/api/repair_request_information/add?",
      url_set: "~/api/repair_request_information/set?",
      url_get_obj: "~/api/repair_request_information/get_obj?",
      url_upload: "~/api/repair_request_information/upload?",

      query: {
        "repair_request_information_id": 0,
      },

      form: {
            "owner_user": 0, // 业主用户
                    "problems_reported_for_repair":  '', // 报修问题
                    "site_photos":  '', // 现场照片
                    "repair_location":  '', // 报修地点
                    "problem_description":  '', // 问题描述
                            "examine_state": "未审核",
                    "examine_reply": "",
                    "repair_request_information_id": 0, // ID
                
              },
          disabledObj:{
                        "owner_user_isDisabled": false,
                                "problems_reported_for_repair_isDisabled": false,
                                "site_photos_isDisabled": false,
                                "repair_location_isDisabled": false,
                                "problem_description_isDisabled": false,
                                  },
                                // 用户列表
            list_user_owner_user: [],
                        // 用户组
            group_user_owner_user: "",
                                                                                                                      list_examine_state:[{value:"未审核",text:"未审核"},{value:"已通过",text:"已通过"},{value:"未通过",text:"未通过"}],
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
            url: _self.$fullUrl('/api/repair_request_information/upload?'),
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
            url: _self.$fullUrl('/api/repair_request_information/upload?'),
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
    async get_list_user_owner_user() {
      // if(this.user_group !== "管理员" && this.form["owner_user"] === 0) {
      //     this.form["owner_user"] = this.user.user_id;
      // }
      var json = await this.$get("~/api/user/get_list?user_group=居民用户");
      if(json.result && json.result.list){
        json.result.list.map((o) => this.list_user_owner_user.push({value:o.user_id,text:o.nickname + '-' + o.username}));
      }
      else if(json.error){
        console.error(json.error);
      }
    },
            /**
     * 获取居民用户用户组
     */
    async get_group_user_owner_user() {
      this.form["owner_user"] = this.user.user_id;
      var json = await this.$get("~/api/user_group/get_obj?name=居民用户");
      if(json.result && json.result.obj){
        this.group_user_owner_user = json.result.obj;
        this.get_user_session_owner_user(this.form['owner_user'])
      }
      else if(json.error){
        console.error(json.error);
      }
    },
    get_user_session_owner_user(id){
      var _this = this;
      var user_id = {"user_id":id}
      var url = "~/api/"+_this.group_user_owner_user.source_table+"/get_obj?"
      this.$get(url, user_id, function(res) {
        if (res.result && res.result.obj) {
          var arr = []
          for (let key in res.result.obj) {
            arr.push(key)
          }
          var arrForm = []
          for (let key in _this.form) {
            arrForm.push(key)
          }
          for (var i=0;i<arr.length;i++){
            if (arr[i]!=='examine_state' && arr[i]!=='examine_reply') {
              for (var j = 0; j < arrForm.length; j++) {
                if (arr[i] === arrForm[j]) {
                  if (arr[i] !== "owner_user") {
                    _this.form[arrForm[j]] = res.result.obj[arr[i]]
                    _this.disabledObj[arrForm[j] + '_isDisabled'] = true
                    break;
                  } else {
                    _this.disabledObj[arrForm[j] + '_isDisabled'] = true
                  }
                }
              }
            }
          }
        }
      });
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
        bl = this.$check_action('/repair_request_information/table','add');
        console.log(bl ? "你有表格添加权限视作有添加权限" : "你没有表格添加权限");
      }
      if(!bl){
        bl = this.$check_action('/repair_request_information/table','set');
        console.log(bl ? "你有表格添加权限视作有修改权限" : "你没有表格修改权限");
      }
      if(!bl){
        bl = this.$check_action('/repair_request_information/view','add');
        console.log(bl ? "你有视图添加权限视作有添加权限" : "你没有视图添加权限");
      }
      if(!bl){
        bl = this.$check_action('/repair_request_information/view','set');
        console.log(bl ? "你有视图修改权限视作有修改权限" : "你没有视图修改权限");
      }
      if(!bl){
        bl = this.$check_action('/repair_request_information/view','get');
        console.log(bl ? "你有视图查询权限视作有查询权限" : "你没有视图查询权限");
      }

      console.log(bl ? "具有当前页面的查看权，请注意这不代表你有字段的查看权" : "无权查看当前页，请注意即便有字段查询权限没有页面查询权限也不行");

      return bl;
    },

  },
  created() {
            this.get_list_user_owner_user();
            this.get_group_user_owner_user();
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
