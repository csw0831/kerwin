<template>
  <div class="layout-container">
    <public-header
      title="升级包管理"
      :breadcrumbs="['运维管理', '终端管理', '升级包管理']"
    ></public-header>
    <el-row class="row-bg">
      <el-col :span="12">
        <el-button type="primary" plain @click="addTerminal"
          >添加升级包</el-button
        >
      </el-col>
      <el-col :span="12" class="el_col_right">
        <div>
          <span>升级包类别:</span>
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
            placeholder="请输入升级包名称/版本号搜索"
            @keyup.native.enter="searchOperator"
            class="order-input"
            :search="search"
          ></el-input>
          <el-button type="primary" @click="searchOperator">搜索</el-button>
        </div>
      </el-col>
    </el-row>
    <el-table :data="operatorList" border>
      <el-table-column label="升级包名称" prop="hostId"></el-table-column>
      <el-table-column label="版本号" prop="name"></el-table-column>
      <el-table-column label="升级包类别" prop="createdAt"></el-table-column>
      <el-table-column label="versioncode" prop="description"></el-table-column>
      <el-table-column label="下载地址" prop="description"></el-table-column>
      <el-table-column label="更新描述" prop="description"></el-table-column>
      <el-table-column label="包类型" prop="description"></el-table-column>
      <el-table-column label="是否有效" prop="description"></el-table-column>
      <el-table-column label="添加者" prop="description"></el-table-column>
      <el-table-column label="添加时间" prop="description"></el-table-column>

      <el-table-column label="操作" width="180">
        <template scope="scope">
          <el-button type="text" size="small" @click="manageOperator(scope.row)"
            >编辑</el-button
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

    <!-- 添加升级包版本 -->
    <el-dialog
      v-model="dialogVisible"
      title="添加版本"
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
        <el-form-item label="升级包名称:" prop="upgradePatchName">
          <el-input
            v-model="form.upgradePatchName"
            placeholder="请输入升级包名称,不超过30字符"
          ></el-input>
        </el-form-item>
        <el-form-item label="导入终端:" prop="uploading" class="wear_span">
          <el-upload
            class="upload-demo upload_demo"
            action="https://jsonplaceholder.typicode.com/posts/"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :before-remove="beforeRemove"
            :before-upload="onBeforeUpload"
            multiple
            accept="image/jpeg,image/gif,image/png"
            :limit="3"
            :on-exceed="handleExceed"
            :file-list="fileList"
            :data="uploadData"
          >
            <!--multiple 允不允许上传多个  -->
            <el-button size="small" type="primary">点击上传</el-button>
            <!-- <div slot="tip" class="el-upload__tip">请上传图片格式文件</div> -->
          </el-upload>
          <span class="akpName">村委APK</span>
        </el-form-item>
        <el-form-item label="更新描述:" prop="description">
          <el-input
            resize="none"
            type="textarea"
            v-model="form.description"
            height="200px"
            :rows="4"
            :maxlength="200"
            placeholder="不超过200字符,不会在界面上显示"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="beforClose('ruleForm')">取 消</el-button>
        <el-button type="primary" @click="submitDatas('ruleForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 编辑的弹框 -->
    <el-dialog
      v-model="dialogVisible3"
      size="tiny"
      title="编辑版本"
      width="60%"
      :before-close="beforClose"
    >
      <el-form
        :model="form"
        ref="ruleForm1"
        label-position="right"
        label-width="100px"
        class="demo-form-inline upgradePatchSpan"
        :rules="rules"
      >
        <el-form-item label="升级包名称:" prop="upgradePatch">
          <el-input v-model="form.upgradePatch"></el-input>
        </el-form-item>
        <el-form-item label="升级包类别:">
          <span>系统包</span>
        </el-form-item>
        <el-form-item label="包名:">
          <span>com.sunight.ll</span>
        </el-form-item>
        <el-form-item label="版本号:">
          <span class="versionNumber">3.1.000.002</span>
        </el-form-item>
        <el-form-item label="下载地址:">
          <span>http://195.168.88/.com</span>
        </el-form-item>
        <el-form-item label="更新描述:">
          <el-input
            resize="none"
            type="textarea"
            v-model="form.description"
            height="200px"
            :rows="4"
            placeholder="不超过200字符,不会在界面上显示"
            :maxlength="200"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="beforClose">取 消</el-button>
        <el-button type="primary" @click="submitData('ruleForm1')"
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
    return {
      operatorList: [],
      limit: 10,
      currentPage: 1,
      total: 0,
      search: "",
      dialogVisible: false, //添加终端隐藏
      dialogVisible3: false, //详情
      form: {
        //添加版本的表单
        upgradePatchName: "", //[添加]升级包名称
        description: "", //更新描述
        upgradePatch: "", //[编辑弹框]升级包名称
      },
      //表单规则
      rules: {
        upgradePatchName: [
          {
            required: true,
            message: "升级包名称不能为空",
            trigger: "blur",
          },
          {
            min: 0,
            max: 30,
            message: "长度在 0 到 30 个字符",
            trigger: "blur",
          },
        ],
        upgradePatch: [
          {
            required: true,
            message: "升级包名称不能为空",
            trigger: "blur",
          },
          {
            min: 0,
            max: 30,
            message: "长度在 0 到 30 个字符",
            trigger: "blur",
          },
        ],
        uploading: [
          {
            required: true,
            message: "升级包不能为空",
            trigger: "change",
          },
        ],
        description: [
          { required: true, message: "更新描述不能为空", trigger: "blur" },
          {
            min: 0,
            max: 100,
            message: "长度在 0 到 100 个字符",
            trigger: "blur",
          },
        ],
      },
      platform: "", //搜索
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
      fileList: [], //导入apk
      uploadData: {
        dataType: "0",
        oldFilePath: "",
      },
    };
  },
  methods: {
    //关闭升级包
    beforClose(fromName) {
      this.closeOrclear();
      if (fromName == "ruleForm") {
        this.$nextTick(() => {
          this.$refs[fromName].resetFields(); // this.$refs.form.resetFields();
        });
      }
    },
    //清空表单数据并关闭
    closeOrclear() {
      this.form = {
        upgradePatchName: "", //[添加]升级包名称
        description: "", //更新描述
        upgradePatch: "", //[编辑弹框]升级包名称
      };
      this.dialogVisible = false;
      this.dialogVisible3 = false;
    },
    //导入终端的
    onBeforeUpload(file) {
      const isIMAGE = file.type === "image/jpeg" || "image/gif" || "image/png";
      const isLt1M = file.size / 1024 / 1024 < 1;

      if (!isIMAGE) {
        this.$message.error("上传文件只能是图片格式!");
      }
      if (!isLt1M) {
        this.$message.error("上传文件大小不能超过 1MB!");
      }
      return isIMAGE && isLt1M;
    },
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
    //编辑的确定按钮
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
    //添加版本的确定按钮
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

    //删除
    del() {
      this.$confirm("此操作将永久删除该升级包, 是否继续?", "提示", {
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
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    //添加升级包
    addTerminal() {
      this.dialogVisible = true;
    },
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
    openCboss(url) {
      window.open(url);
    },
    searchOperator() {
      // this.$route.query._page == 1? this.getOperators() : (this.currentPage = 1)
      this.currentPage = 1;
      this.getOperators();
    },
  },
  watch: {
    $route() {
      // this.getOperators()
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
.upgradePatchSpan {
  .versionNumber {
    color: blue;
    text-decoration: underline;
  }
  span {
    line-height: 36px;
  }
}
</style>
<style lang="less">
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
.upload_demo .el-upload {
  display: flex;
  width: 40% !important;
}
.wear_span .el-form-item__content {
  display: flex;
  .akpName {
    margin-left: 20px;
    line-height: 28px;
  }
}
</style>
