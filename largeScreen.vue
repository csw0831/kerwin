<template>
  <div class="layout-container">
    <public-header
      title="便民大屏"
      :breadcrumbs="['运维管理', '终端管理', '便民大屏']"
    ></public-header>
    <el-row class="row-bg">
      <el-col :span="12">
        <el-button type="primary" plain @click="addTerminal"
          >添加终端</el-button
        >
        <el-button type="primary" plain @click="toLead">批量导入</el-button>
      </el-col>
      <el-col :span="12" class="el_col_right">
        <div>
          <span>所属业务平台:</span>
          <el-select v-model="platform" placeholder="请选择">
            <el-option
              v-for="item in platform_options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            >
            </el-option>
          </el-select>
          <el-input
            v-model="search"
            placeholder="请输入终端名称/终端号搜索"
            @keyup.native.enter="searchOperator"
            class="order-input"
            :search="search"
          ></el-input>
          <el-button type="primary" @click="searchOperator">搜索</el-button>
        </div>
      </el-col>
    </el-row>
    <el-table :data="operatorList" border>
      <el-table-column label="终端名称" prop="hostId"></el-table-column>
      <el-table-column label="终端ID" prop="name"></el-table-column>
      <el-table-column label="mac地址" prop="createdAt"></el-table-column>
      <el-table-column label="系统版本" prop="description"></el-table-column>
      <el-table-column label="所属平台" prop="description"></el-table-column>
      <el-table-column label="在线情况" prop="description"></el-table-column>

      <el-table-column label="操作" width="180">
        <template scope="scope">
          <el-button type="text" size="small" @click="manageOperator(scope.row)"
            >详情</el-button
          >
          <el-button type="text" size="small" @click="relieve"
            >解除绑定</el-button
          >
          <el-button type="text" size="small" @click="del">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 这个是分页 -->
    <div class="table-pagination">
      <el-pagination
        :current-page="currentPage"
        :page-sizes="[10, 20, 30, 40, 50, 100, 500, 1000]"
        :page-size="limit"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        @current-change="handleCurrentPageChanged"
        @size-change="handlePageSizeChanged"
      >
      </el-pagination>
    </div>

    <!-- 添加终端弹框 -->
    <el-dialog
      v-model="dialogVisible"
      title="添加终端"
      size="tiny"
      @close="beforClose('ruleForm')"
    >
      <el-form
        :model="form"
        ref="ruleForm"
        label-position="right"
        label-width="100px"
        class="demo-form-inline"
        :rules="rules"
      >
        <el-form-item label="终端ID:" prop="terminalid" class="noneIcon">
          <el-input
            v-model.trim="form.terminalid"
            placeholder="数字、字母组成的终端ID"
          ></el-input>
        </el-form-item>
        <el-form-item label="终端名称:" prop="terminalName">
          <el-input
            v-model.trim="form.terminalName"
            placeholder="请输入终端名称,不超过30字符"
          ></el-input>
        </el-form-item>
        <el-form-item
          class="phonenum"
          label="所属业务平台:"
          prop="businessPlatform"
        >
          <el-select
            v-model="form.businessPlatform"
            placeholder="请选取终端数所属页面平台"
          >
            <el-option
              v-for="item in platform_options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="终端标签:">
          <!-- <el-input v-model="form.memo"></el-input> -->
          <el-select
            popper-class="myselect"
            :popper-append-to-body="false"
            v-model="form.memo"
            clearable
            multiple
            filterable
            allow-create
            default-first-option
            placeholder="输入标签后按'enter'键添加"
            class="memo"
          >
            <el-option
              v-for="item in knowledge"
              :key="item.value"
              :label="item.label"
              :value="item.label"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="beforClose('ruleForm')">取 消</el-button>
        <el-button type="primary" @click="submitData('ruleForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 导入终端的弹框 -->
    <el-dialog v-model="dialogVisible2" title="添加终端" size="tiny">
      <el-form
        :model="form"
        ref="ruleForm1"
        label-position="right"
        label-width="100px"
        class="demo-form-inline"
        :rules="rules"
      >
        <p class="margin_p">
          点击<span class="blueColor">下载模板表格</span
          >,建议每次上传不超过1000条数据,未导入完成请勿关闭弹窗
        </p>
        <el-form-item label="导入终端:">
          <el-upload
            class="upload-demo"
            action="https://jsonplaceholder.typicode.com/posts/"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :before-remove="beforeRemove"
            multiple
            :limit="3"
            :on-exceed="handleExceed"
            :file-list="fileList"
          >
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible2 = false">取 消</el-button>
        <el-button type="primary" @click="submitDatas('ruleForm1')"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 详情的弹框 -->
    <el-dialog
      v-model="dialogVisible3"
      size="tiny"
      class="detailPage"
      width="60%"
      :show-close="false"
    >
      <div class="close" @click="cloneDialog">
        <i class="el-icon-close"></i>
      </div>
      <el-form
        :model="form"
        ref="ruleForm1"
        label-position="right"
        label-width="100px"
        class="demo-form-inline"
        :rules="rules"
      >
        <el-tabs v-model="activeName" @tab-click="handleClick" class="message">
          <el-tab-pane label="基本信息" name="first">
            <el-form-item label="终端号:">
              <span>1000230</span>
            </el-form-item>
            <el-form-item label="终端名称:" prop="terminalName2">
              <el-input v-model.trim="form.terminalName2"></el-input>
            </el-form-item>
            <el-form-item label="mac地址:">
              <span>09:Y6:09:D4:T5</span>
            </el-form-item>
            <el-form-item label="系统版本:">
              <span>Android1.0.009</span>
            </el-form-item>
            <el-form-item label="ip地址:">
              <span>172.10.23.256</span>
            </el-form-item>
            <el-form-item label="分辨率:">
              <span>1920x1080</span>
            </el-form-item>
            <el-form-item label="存储空间:">
              <div class="schedule">
                <el-progress :percentage="50" :show-text="false"></el-progress>
                <span>12.8G/100G</span>
              </div>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="软件信息" name="second" class="message">
            <el-table :data="tableData" border style="width: 100%">
              <el-table-column prop="date" label="应用名称" width="100">
              </el-table-column>
              <el-table-column prop="name" label="文件名" width="90">
              </el-table-column>
              <el-table-column prop="address" label="版本号" width="170">
              </el-table-column>
              <el-table-column prop="address" label="更新时间" width="175">
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cloneDialog">取 消</el-button>
        <el-button type="primary" @click="submitData('ruleForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>
<script>
import PublicHeader from "@/components/PublicHeader";
import hotel from "@/stores/hotelStore";

export default {
  components: {
    PublicHeader,
  },
  data() {
    var terminalidRules = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("终端ID不能为空"));
      }
      if (!/^[A-Za-z0-9]+$/.test(value)) {
        return callback(new Error("终端ID由数字、字母组成"));
      }
    };

    return {
      operatorList: [],
      limit: 10,
      currentPage: 1,
      total: 0,
      search: "",
      dialogVisible: false, //添加终端隐藏
      dialogVisible2: false, //导入终端
      dialogVisible3: false, //详情
      form: {
        //添加终端的表单
        terminalid: "", //终端id
        terminalName: "", //终端名称
        businessPlatform: "", //所属业务平台
        memo: "", //终端标签
        terminalName2: "", //详情页基本信息的终端名称
      },
      rules: {
        //表单规则
        terminalid: [
          { required: true, validator: terminalidRules, trigger: "blur" },
        ],
        terminalName: [
          { required: true, message: "终端名称不能为空", trigger: "blur" },
          {
            min: 0,
            max: 30,
            message: "长度在 0 到 30 个字符",
            trigger: "blur",
          },
        ],
        terminalName2: [
          { required: true, message: "终端名称不能为空", trigger: "blur" },
        ],
        businessPlatform: [
          { required: true, message: "请选择所属业务平台", trigger: "blur" },
        ],
      },
      platform: "", //搜索的平台单选项(上面是弹框的)
      platform_options: [
        //平台选项
        {
          value: "选项1",
          label: "黄金糕",
        },
        {
          value: "选项2",
          label: "双皮奶",
        },
        {
          value: "选项3",
          label: "蚵仔煎",
        },
        {
          value: "选项4",
          label: "龙须面",
        },
        {
          value: "选项5",
          label: "北京烤鸭",
        },
      ],
      fileList: [], //上传图片
      activeName: "first", //点击详情默认选中第一个
      tableData: [
        {
          date: "2016-05-02",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄",
        },
        {
          date: "2016-05-04",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1517 弄",
        },
        {
          date: "2016-05-01",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1519 弄",
        },
        {
          date: "2016-05-03",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1516 弄",
        },
      ],
      knowledge: [], //存放标签的
    };
  },
  methods: {
    //点击关闭按钮[添加终端]
    beforClose(fromName) {
      this.closeOrclear();
      console.log(fromName);
      if (fromName) {
        this.$nextTick(() => {
          this.$refs[fromName].resetFields(); // this.$refs.form.resetFields();
        });
      }
    },
    //清空表单数据并关闭
    closeOrclear() {
      this.form = {
        terminalid: "", //终端id
        terminalName: "", //终端名称
        businessPlatform: "", //所属业务平台
        memo: "", //终端标签
        terminalName2: "", //详情页基本信息的终端名称
      };
      this.dialogVisible = false;
    },
    //关闭详情
    cloneDialog() {
      this.dialogVisible3 = false;
      this.form.terminalName2 = "";
    },
    //切换详情
    handleClick(tab, event) {
      console.log(tab, event, "122");
    },
    //导入终端的[文件上传]
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${
          files.length + fileList.length
        } 个文件`
      );
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`);
    },
    //导入终端的确定按钮
    async submitDatas(ruleForm1) {
      console.log(this.fileList, "this.fileList");
      if (this.fileList.length == 0) {
        this.$message("请上传文件");
      }
      // console.log("导入终端");
      // this.$refs[ruleForm1].validate(async (valid2) => {
      //   if (!valid2) {
      //     return false;
      //   }
      // if (this.form.terminalid.trim() === "") {
      //   this.$message.error("不能为空");
      //   return;
      // }
      // const params = JSON.parse(JSON.stringify(this.form));
      // params.groupType = Number(this.form.groupType);
      // const res = await this.AX("hotel/saveGroup", params, "post");
      // if (res.code === 0) {
      //   this.fetch();
      //   this.$message.success("操作成功");
      //   setTimeout(() => {
      //     this.dialogVisible = false;
      //   }, 300);
      // } else {
      //   this.$message.error(res.message);
      // }
      // });
    },
    //添加终端
    async submitData(ruleForm) {
      this.$refs[ruleForm].validate(async (valid) => {
        if (!valid) {
          return false;
        }
        // if (this.form.terminalid.trim() === "") {
        //   this.$message.error("不能为空");
        //   return;
        // }
        // const params = JSON.parse(JSON.stringify(this.form));
        // params.groupType = Number(this.form.groupType);
        // const res = await this.AX("hotel/saveGroup", params, "post");
        // if (res.code === 0) {
        //   this.fetch();
        //   this.$message.success("操作成功");
        //   setTimeout(() => {
        //     this.dialogVisible = false;
        //   }, 300);
        // } else {
        //   this.$message.error(res.message);
        // }
      });
    },
    //解除绑定
    relieve() {
      console.log("解除绑定");
    },
    //删除
    del() {
      this.$confirm("此操作将永久删除该终端, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message("已取消");
        });
    },
    //添加终端弹窗出现
    addTerminal() {
      this.dialogVisible = true;
    },
    //导入终端弹窗
    toLead() {
      this.dialogVisible2 = true;
    },
    // 详情弹窗
    manageOperator(row) {
      this.dialogVisible3 = true;
    },
    getOperators() {
      return hotel
        .getHotelList({
          limit: this.limit,
          page: this.currentPage,
          search: this.search,
          hotelType: "1",
        })
        .then((res) => {
          this.operatorList = res.data.docs;
          this.total = res.data.total;

          sessionStorage.setItem(
            "operatorList",
            JSON.stringify({
              currentPage: this.currentPage,
              pageSize: this.limit,
              search: this.search,
            })
          );
        });
    },
    handleCurrentPageChanged(page) {
      this.currentPage = page;
      this.getOperators();
      // this.$router.replace({
      //   path: '/hotel/operator-list',
      //   query: { _page: page, _limit: this.limit }
      // })
    },
    handlePageSizeChanged(size) {
      this.limit = size;
      this.getOperators();
      // this.$router.replace({
      //   path: '/hotel/operator-list',
      //   query: { _page: this.currentPage, _limit: size }
      // })
    },
    //无用,先注释 以后再说
    // openCboss(url) {
    //   window.open(url);
    // },
    searchOperator() {
      // this.$route.query._page == 1? this.getOperators() : (this.currentPage = 1)
      this.currentPage = 1;
      this.getOperators();
    },
  },
  watch: {
    $route(to, from) {
      // 做一些路由变化的响应
      // this.loading = true;
      // this.fetchLink();
      console.log(to, from, "1111");
    },
  },
  created() {
    if (sessionStorage.getItem("operatorList")) {
      const pageData = JSON.parse(sessionStorage.getItem("operatorList"));
      this.currentPage = pageData.currentPage || 1;
      this.limit = pageData.pageSize || 10;
      this.search = pageData.search;
    }
    this.getOperators();
  },
  mounted() {},
  beforeRouteUpdate(to, from, next) {
    if (to.path === "/hotel/operator-list" && to.query._page) {
      next();
    } else {
      return;
    }
  },
};
</script>
<style lang="less" scoped>
.row-bg {
  background: #f9fafc;
  padding: 30px 15px;
  margin: 22px 0;
}

.order-input {
  display: inline-block;
  width: 250px;
}
//kerwin
.el_col_right {
  text-align: right;
}
.demo-form-inline {
  .margin_p {
    margin: 20px 0;
    .blueColor {
      color: blue;
    }
  }
}
</style>
<style lang="less">
.phonenum > .el-form-item__label {
  white-space: nowrap;
}
.el-upload {
  width: 12% !important;
}
.detailPage {
  .close {
    position: absolute;
    right: 20px;
    top: 10px;
    z-index: 10000000000000000000000;
  }
  .el-dialog__header {
    padding: 0px 20px 0;
  }

  .el-dialog__body {
    padding: 0px 20px 20px;
    color: #48576a;
    font-size: 14px;
  }
}

.message {
  .el-tabs__active-bar {
    display: none;
  }
  .el-tabs__item {
    line-height: 48px;
    font-size: 16px;
  }
}
.schedule .el-progress {
  width: 81%;
  display: inline-block;
}
.noneIcon .el-input__icon {
  display: none;
}
.memo {
  width: 100%;
  .el-icon-caret-top {
    display: none;
  }
  .el-input__inner {
    padding-right: 10px;
  }
}
.myselect {
  display: none;
}
</style>
