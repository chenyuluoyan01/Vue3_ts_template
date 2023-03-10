<template>
	<el-config-provider :locale="zhCn">
		<router-view />
		<CloseFull />
	</el-config-provider>
</template>

<script setup lang="ts" name="app">
import { defineAsyncComponent, computed, ref, onBeforeMount, onMounted, nextTick } from 'vue';

import { storeToRefs } from 'pinia';
import { useTagsViewRoutes } from '/@/stores/tagsViewRoutes';
import { useThemeConfig } from '/@/stores/themeConfig';
import { Local, Session } from '/@/utils/storage';
import setIntroduction from '/@/utils/setIconfont';
import { useChangeColor } from '/@/utils/theme';
import zhCn from "element-plus/lib/locale/lang/zh-cn"

// 引入组件
const CloseFull = defineAsyncComponent(() => import('/@/layout/navBars/breadcrumb/closeFull.vue'));
const { getLightColor } = useChangeColor();

// 定义变量内容
const stores = useTagsViewRoutes();
const storesThemeConfig = useThemeConfig();
const { themeConfig } = storeToRefs(storesThemeConfig);

// 设置初始化，防止刷新时恢复默认
onBeforeMount(() => {
	// 设置批量第三方 icon 图标
	setIntroduction.cssCdn();
	// 设置批量第三方 js
	setIntroduction.jsCdn();
});
// 获取布局配置信息
const getThemeConfig = computed(() => {
	return themeConfig.value;
});

// 设置布局切换函数
const initLayoutChangeFun = () => {
	onBgColorPickerChange('menuBar');
	onBgColorPickerChange('menuBarColor');
	onBgColorPickerChange('menuBarActiveColor');
};

const onBgColorPickerChange = (bg: string) => {
	document.documentElement.style.setProperty(`--next-bg-${bg}`, themeConfig.value[bg]);
	if (bg === 'menuBar') {
		document.documentElement.style.setProperty(`--next-bg-menuBar-light-1`, getLightColor(getThemeConfig.value.menuBar, 0.05));
	}
	onTopBarGradualChange();
	onMenuBarGradualChange();
};
// 2、菜单 / 顶栏 --> 顶栏背景渐变
const onTopBarGradualChange = () => {
	setGraduaFun('.layout-navbars-breadcrumb-index', getThemeConfig.value.isTopBarColorGradual, getThemeConfig.value.topBar);
};
// 2、菜单 / 顶栏 --> 菜单背景渐变
const onMenuBarGradualChange = () => {
	setGraduaFun('.layout-container .el-aside', getThemeConfig.value.isMenuBarColorGradual, getThemeConfig.value.menuBar);
};

const setGraduaFun = (el: string, bool: boolean, color: string) => {
	setTimeout(() => {
		let els = document.querySelector(el);
		if (!els) return false;
		document.documentElement.style.setProperty('--el-menu-bg-color', document.documentElement.style.getPropertyValue('--next-bg-menuBar'));
		if (bool) els.setAttribute('style', `background:linear-gradient(to bottom left , ${color}, ${getLightColor(color, 0.6)}) !important;`);
		else els.setAttribute('style', ``);
		setLocalThemeConfig();
	}, 200);
};

// 存储布局配置
const setLocalThemeConfig = () => {
	Local.remove('themeConfig');
	Local.set('themeConfig', getThemeConfig.value);
};

// 页面加载时
onMounted(() => {
	nextTick(() => {
		initLayoutChangeFun()
		// 获取缓存中的布局配置
		if (Local.get('themeConfig')) {
			storesThemeConfig.setThemeConfig({ themeConfig: Local.get('themeConfig') });
			document.documentElement.style.cssText = Local.get('themeConfigStyle');
		}
		// 获取缓存中的全屏配置
		if (Session.get('isTagsViewCurrenFull')) {
			stores.setCurrenFullscreen(Session.get('isTagsViewCurrenFull'));
		}
		setTimeout(() => {
			initLayoutChangeFun()
		}, 100)
	});
});
</script>
