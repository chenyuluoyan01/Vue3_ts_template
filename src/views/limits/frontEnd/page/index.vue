<template>
	<div class="layout-pd">
		<el-alert
			:title="`当前用户页面权限：[${userInfos.roles}]，当前用户按钮权限：[${userInfos.authBtnList}]`"
			type="success"
			:closable="false"
			class="mt15"
		></el-alert>
		<el-card shadow="hover" header="切换用户演示，前端控制不同用户显示不同页面、按钮权限" class="mt15">
			<el-radio-group v-model="userAuth" size="default" @change="onRadioChange">
				<el-radio-button label="admin"></el-radio-button>
				<el-radio-button label="common"></el-radio-button>
			</el-radio-group>
		</el-card>
	</div>
</template>

<script setup lang="ts" name="limitsFrontEndPage">
import { onMounted, ref } from 'vue';
import Cookies from 'js-cookie';
import { storeToRefs } from 'pinia';
import { useUserInfo } from '/@/stores/userInfo';
import { frontEndsResetRoute, setAddRoute, setFilterMenuAndCacheTagsViewRoutes } from '/@/router/frontEnd';

// 定义变量内容
const storesUserInfo = useUserInfo();
const { userInfos } = storeToRefs(storesUserInfo);
const userAuth = ref('');

// 初始化用户权限
const initUserAuth = () => {
	userAuth.value = userInfos.value.roles[0];
};
// 用户权限改变时
const onRadioChange = async () => {
	// 模拟数据
	frontEndsResetRoute();
	Cookies.set('userName', userAuth.value);
	// 模拟切换不同权限用户
	await storesUserInfo.setUserInfos();
	await setAddRoute();
	setFilterMenuAndCacheTagsViewRoutes();
};
// 页面加载时
onMounted(() => {
	initUserAuth();
});
</script>
