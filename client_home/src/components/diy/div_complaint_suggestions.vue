<template>
	<view class="diy_details diy_div_complaint_suggestions">
		<view v-if="Object.keys(obj).length!==0" class="warp">
			<view class="container">
				<view class="row">
					<view v-if="$check_field('get','owner')" class="col-12 col-md-6">
						<view class="diy_title">
							<span>业主:</span>
						</view>
							<view class="diy_field diy_uid">
							<text>
								{{ get_user_owner(obj['owner']) }}
							</text>
						</view>
						</view>
					<view v-if="$check_field('get','room_number')" class="col-12 col-md-6">
						<view class="diy_title">
							<span>房号:</span>
						</view>
							<view class="diy_field diy_text">
							<text>
								{{obj["room_number"]}}
							</text>
						</view>
						</view>
					<view v-if="$check_field('get','complaint_issues')" class="col-12 col-md-6">
						<view class="diy_title">
							<span>投诉问题:</span>
						</view>
							<view class="diy_field diy_desc">
							<text>
								{{ obj["complaint_issues"] }}
							</text>
						</view>
						</view>
					<view v-if="$check_field('get','suggested_feedback')" class="col-12 col-md-6">
						<view class="diy_title">
							<span>建议反馈:</span>
						</view>
							<view class="diy_field diy_desc">
							<text>
								{{ obj["suggested_feedback"] }}
							</text>
						</view>
						</view>
					<view v-if="$check_field('get','related_pictures')" class="col-12 col-md-6">
						<view class="diy_title">
							<span>相关图片:</span>
						</view>
							<view class="diy_field diy_img">
							<image style="width:100%;height:auto;" :src="$fullUrl(obj['related_pictures'])" mode="widthFix"></image>
						</view>
						</view>
				</view>
			</view>
		</view>

	</view>
</template>

<script>
	import mixin from "@/libs/mixins/component.js";
	export default {
		mixins: [mixin],
		props: {
			obj_goods: {
				type: Object,
				default(){
					return {}
				}
			},
			query:{
				type: Object,
				default(){
					return {
						complaint_suggestions_id: 0
					}
				}
			}
		},
		data() {
			return {
				obj_goods_type: {},
				obj: {},
					// 用户列表
				list_user_owner: [],
								}
		},
		methods: {
			async get_obj_goods_type() {
				var res = await this.$get("~/api/goods_type/get_obj", {
					name: this.obj_goods.type
				})

				if (res.result.obj) {
					this.obj_goods_type = res.result.obj
				} else {
					console.log("没有请求到商品分类");
				}
			},
			async get_obj_by_goods() {
				var {
					source_table,
					source_field
				} = this.obj_goods_type
				var {
					source_id
				} = this.obj_goods
				var query = {}
				query[source_field] = source_id
				this.$get("~/api/" + source_table + "/get_obj", {}, (res) => {
					if (res.result.obj) {
						this.obj = res.result.obj
					} else {
						console.log("没有请求到商品分类");
					}
				})
			},
			async get_obj_by_id(){
				var res = await this.$get("~/api/complaint_suggestions/get_obj", {
					complaint_suggestions_id:this.query.complaint_suggestions_id
				})

				if (res.result && res.result.obj) {
					this.obj = res.result.obj
				} else {
					console.log("没有请求到商品分类");
				}
			},
				/**
			 * 获取居民用户用户列表
			 */
			async get_list_user_owner() {
				var json = await this.$get("~/api/user/get_list?user_group=居民用户");
				if(json.result && json.result.list){
					this.list_user_owner = json.result.list;
				}
				else if(json.error){
					console.error(json.error);
				}
			},
			get_user_owner(id){
				let obj = this.list_user_owner;
				let ret = "";
				for(let i=0;i<obj.length;i++){
					if(obj[i].user_id==id){
						ret = obj[i].nickname+"-"+obj[i].username;
					}
				}
				return ret;
			},
							},
		created() {
				this.get_list_user_owner();
							},
		async onLoad(){
			if (Object.keys(this.obj_goods).length !== 0) {
				await this.get_obj_goods_type();
				await this.get_obj_by_goods();
			} else if (this.query["complaint_suggestions_id"] !==0) {
				await this.get_obj_by_id();
			}
		}
	}
</script>

<style scoped>
	.div_complaint_suggestions_details {}

	.details {
		background-color: #fff;
		margin-bottom: 0.5rem;
		padding: 1rem;
		font-size: 10px;
	}

	.item {
		display: flex;
		padding: 0.2rem 0;
		border-bottom: 1px solid #eee;
	}

	.left_text {
		flex: 0 0 25%;
	}

	.right_text {
		flex: 0 0 75%;
	}
	.edit_nav{
		text-align: center;
		background-color: #fff;
		padding: 0.3rem;
		margin: 0.1rem 1rem;
		border: 1px solid #eee;
		border-radius: 0.5rem;
	}
</style>
