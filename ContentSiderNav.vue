<template>
  <el-menu
    :router="true"
    class="side-nav"
    @open="handleOpen"
    @close="handleClose"
    :default-active="submenuIndex"
    @select="handleSelect"
  >
    <el-submenu
      :index="index1 + ''"
      v-for="(item, index1) in ifShowMenuItem"
      :key="index1"
      v-if="item.isVisible === 1"
    >
      <template slot="title">
        <VIcon class="iconfont-1" :type="item.type" />{{
          item.menuName
        }}</template
      >
      <el-menu-item
        class="side-nav-item"
        :index="item2.url"
        v-for="(item2, index2) in item.children"
        :key="index2"
        v-if="!item2.children && item2.isVisible === 1"
      >
        <VIcon class="iconfont-1" :type="item2.type" />{{ item2.menuName }}
      </el-menu-item>
      <el-submenu
        :index="index1 + '-' + index2"
        v-for="(item2, index2) in item.children"
        :key="index2"
        v-if="item2.children && item2.isVisible === 1"
        class="side-nav-item"
      >
        <template slot="title">
          <VIcon class="iconfont-1" :type="item2.type"></VIcon
          >{{ item2.menuName }}</template
        >
        <el-menu-item
          class="side-nav-item"
          v-for="(item3, index3) in item2.children"
          :key="index3"
          :index="item3.url"
          v-if="item3.isVisible === 1"
        >
          <VIcon class="iconfont-1" :type="item3.type"></VIcon
          >{{ item3.menuName }}</el-menu-item
        >
      </el-submenu>
    </el-submenu>
    <!-- <el-submenu index="6">
            <template slot="title">
                <VIcon class='iconfont-1'
                       type='book' />广告分发管理</template>
            <el-menu-item class='side-nav-item'
                          index="/content/material-audit">
                <VIcon class='iconfont-1'
                       type='bars' />素材审核</el-menu-item>
        </el-submenu> -->
    <!-- <el-submenu index="1">
            <template slot="title">
                <VIcon class='iconfont-1'
                       type='book' />栏目管理</template>
            <el-menu-item class='side-nav-item'
                          index="/content">
                <VIcon class='iconfont-1'
                       type='bars' />栏目下发管理</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="1-2">
                <VIcon class='iconfont-1'
                       type='bars' />栏目审核</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/content/items-store">
                <VIcon class='iconfont-1'
                       type='book' />栏目库</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/content/items-tree-store">
                <VIcon class='iconfont-1'
                       type='sharealt' />栏目树库
            </el-menu-item>

        </el-submenu>
        <el-submenu index="2">
            <template slot="title">
                <VIcon class='iconfont-1'
                       type='hdd' />素材管理</template>
            <el-menu-item class='side-nav-item'
                          index="/content/public-pics-store">
                <VIcon class='iconfont-1'
                       type='picture' />公共图片库</el-menu-item>
        </el-submenu>
        <el-submenu index="3">
            <template slot="title">
                <VIcon class='iconfont-1'
                       type="retweet"></VIcon>本地生活圈</template>
            <el-submenu index="3-1"
                        class='side-nav-item'>
                <template slot="title">
                    <VIcon class='iconfont-1'
                           type="laptop"></VIcon>商户库</template>
                <el-menu-item class='side-nav-item'
                              index="/content/add-merchant">
                    <VIcon class='iconfont-1'
                           type="plus-square-o"></VIcon>添加商户</el-menu-item>
                <el-menu-item class='side-nav-item'
                              index="/content/all-merchant">
                    <VIcon class='iconfont-1'
                           type="calculator"></VIcon>全部商户</el-menu-item>
            </el-submenu>
            <el-submenu index="3-2"
                        class='side-nav-item'>
                <template slot="title">
                    <VIcon class='iconfont-1'
                           type="question-circle-o"></VIcon>
                    商户审核
                    <el-badge :value="uncheckednumIndex"
                              class="item"
                              v-show="uncheckednumIndex>0">
                        <span style="display: inline-block;width: 10px;"></span>
                    </el-badge>
                </template>
                <el-menu-item class='side-nav-item'
                              index="/content/unchecked">
                    <VIcon class='iconfont-1'
                           type="minus-circle-o"></VIcon>待审核</el-menu-item>
                <el-menu-item class='side-nav-item'
                              index="/content/passed">
                    <VIcon class='iconfont-1'
                           type="check-circle-o"></VIcon>已通过</el-menu-item>
                <el-menu-item class='side-nav-item'
                              index="/content/rejected">
                    <VIcon class='iconfont-1'
                           type="close-circle-o"></VIcon>驳回</el-menu-item>
            </el-submenu>
            <el-submenu index="3-3"
                        class='side-nav-item'>
                <template slot="title">
                    <VIcon class='iconfont-1'
                           type="file-ext"></VIcon>商户发布</template>
                <el-menu-item class='side-nav-item'
                              index="/content/publish-management">
                    <VIcon class='iconfont-1'
                           type="filter"></VIcon>发布管理</el-menu-item>
                <el-menu-item class='side-nav-item'
                              index="/content/publish-logs">
                    <VIcon class='iconfont-1'
                           type="tags-o"></VIcon>发布日志</el-menu-item>
            </el-submenu>
            <el-submenu index="3-4"
                        class='side-nav-item'>
                <template slot="title">
                    <VIcon class='iconfont-1'
                           type="exclamation-circle-o"></VIcon>发布情况</template>
                <el-menu-item class='side-nav-item'
                              index="/content/hotel-publish-status">
                    <VIcon class='iconfont-1'
                           type="hotel"></VIcon>酒店发布情况</el-menu-item>
            </el-submenu>
             <el-submenu index="3-5">
                <template slot="title">
                    <Icon class='iconfont-1' type="pie-graph"></Icon>数据统计</template>
                <el-menu-item index="/content/merchant-data">
                    <Icon class='iconfont-1' type="calculator"></Icon>商户数据</el-menu-item>
                <el-menu-item index="/content/online-data">
                    <Icon class='iconfont-1' type="ios-pulse-strong"></Icon>上线数据</el-menu-item>
            </el-submenu>
        </el-submenu>
        <el-submenu index="11">
            <template slot="title">
                <VIcon class='iconfont-1'
                       type='main_menu' />主菜单管理</template>
            <el-menu-item class='side-nav-item'
                          index="/main_menu/material">
                <VIcon class='iconfont-1'
                       type='bars' />主菜单素材</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/main_menu/index">
                <VIcon class='iconfont-1'
                       type='bars' />主菜单管理</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/main_menu/release">
                <VIcon class='iconfont-1'
                       type='bars' />发布管理</el-menu-item>
        </el-submenu>
        <el-submenu index="12">
            <template slot="title">
                <VIcon class='iconfont-1'
                       type='media' />媒资管理</template>
            <el-menu-item class='side-nav-item'
                          index="/media/pending">
                <VIcon class='iconfont-1'
                       type='bars' />待入库</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/media/index">
                <VIcon class='iconfont-1'
                       type='bars' />影片库</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/media/topic">
                <VIcon class='iconfont-1'
                       type='bars' />专题库</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/media/fromlist">
                <VIcon class='iconfont-1'
                       type='bars' />媒资提供方列表</el-menu-item>
        </el-submenu>
        <el-submenu index="13">
            <template slot="title">
                <VIcon class='iconfont-1'
                       type='vod_home' />VOD首页管理</template>
            <el-menu-item class='side-nav-item'
                          index="/vod_home/release_manage">
                <VIcon class='iconfont-1'
                       type='bars' />发布管理</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/vod_home/home_manage">
                <VIcon class='iconfont-1'
                       type='bars' />VOD首页管理</el-menu-item>
            <el-menu-item class='side-nav-item'
                          index="/vod_home/release_log">
                <VIcon class='iconfont-1'
                       type='bars' />首页发布日志</el-menu-item>
        </el-submenu> -->
  </el-menu>
</template>
<script>
import { VIcon } from "../components";
import "../style/index.less";
import { appStore, contentStore } from "../stores";
import { mapGetters, mapActions } from "vuex";
export default {
  data() {
    return {
      ifShowMenuItem: [],
    };
  },
  computed: {
    ...mapGetters({
      submenuIndex: "content/GET_SUB_MENU_INDEX",
      uncheckednumIndex: "content/GET_UNCHECKED_NUM",
    }),
  },
  components: {
    VIcon,
  },
  methods: {
    getAuthority() {
      appStore
        .authority()
        .then((res) => {
          console.log(res);
          this.ifShowMenuItem = res.data.docs.filter(
            (item) => item.menuCode === 120
          )[0].children;
          console.log(this.ifShowMenuItem, "this.ifShowMenuItem");
        })
        .catch((err) => {
          this.$message.error(`${err}`);
        });
    },
    ...mapActions({
      setSubmenuIndex: "content/SET_SUB_MENU_INDEX",
      setUncheckednumIndex: "content/SET_UNCHECKED_NUM",
    }),
    handleOpen(key, keyPath) {},
    handleClose(key, keyPath) {},
    handleSelect(key, keyPath) {
      this.setSubmenuIndex({
        submenuIndex: key,
      });
    },
  },
  created() {
    this.getAuthority();
    contentStore
      .listUncheckMerchantInfo({
        page: 1,
        limit: 10,
      })
      .then((res) => {
        this.setUncheckednumIndex({
          uncheckednumIndex: Number(res.data.total),
        });
      })
      .catch((err) => {
        console.log(`获取数据出错了${err}`);
      });
  },
};
</script>
