<template>
  <div class="layout-container">
    <public-header
      title="升级策略管理"
      :breadcrumbs="['运维管理', '升级管理', '升级策略配置']"
    ></public-header>
    <el-row class="row-bg">
      <el-col :span="12">
        <el-button type="primary" plain @click="addTerminal"
          >创建策略</el-button
        >
      </el-col>
      <el-col :span="12" class="el_col_right">
        <div>
          <el-input
            v-model="search"
            placeholder="请输入策略名称搜索"
            @keyup.native.enter="searchOperator"
            class="order-input"
            :search="search"
          ></el-input>
          <el-button type="primary" @click="searchOperator">搜索</el-button>
        </div>
      </el-col>
    </el-row>
    <el-table :data="operatorList" border>
      <el-table-column label="策略名称" prop="hostId"></el-table-column>
      <el-table-column label="更新后版本" prop="name"></el-table-column>
      <el-table-column label="更新终端类型" prop="createdAt"></el-table-column>
      <el-table-column label="更新数量" prop="description"></el-table-column>

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

    <!-- 添加更新策略 -->
    <el-dialog
      v-model="dialogVisible"
      :title="title"
      size="tiny"
      @close="beforCloses('ruleForm')"
    >
      <el-form
        :model="form"
        ref="ruleForm"
        label-position="right"
        label-width="100px"
        class="demo-form-inline"
        :rules="rules"
      >
        <el-form-item label="策略名称:" prop="upgradePatchName">
          <el-input
            v-model="form.upgradePatchName"
            placeholder="请输入策略名称,不超过30字符"
          ></el-input>
        </el-form-item>
        <el-form-item label="更新后版本:" prop="businessPlatform">
          <el-select v-model="form.businessPlatform">
            <el-option
              v-for="item in platform_options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item
          label="更新终端类型:"
          prop="businessPlatform2"
          class="phonenum"
        >
          <el-select v-model="form.businessPlatform2">
            <el-option
              v-for="item in UpdatedVersionData"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="更新数量:" prop="businessPlatform3">
          <el-select v-model="form.businessPlatform3">
            <el-option
              v-for="item in updateNumData"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              @click.native="changenum(item.value)"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item
          label="选取终端:"
          prop="uploading"
          class="wear_span"
          v-if="this.title === '添加更新策略'"
        >
          <el-upload
            class="upload-demo upload_demo"
            action="https://jsonplaceholder.typicode.com/posts/"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :before-remove="beforeRemove"
            multiple
            :limit="3"
            :on-exceed="handleExceed"
            :file-list="fileList"
          >
            <!--multiple 允不允许上传多个  -->
            <el-button size="small" type="primary">选择终端</el-button>
          </el-upload>
        </el-form-item>
        <el-form-item label="备注:">
          <el-input
            resize="none"
            type="textarea"
            v-model="form.description"
            height="200px"
            :rows="4"
            :maxlength="200"
            placeholder="不超过200字符"
          ></el-input>
        </el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="form.resource">
            <el-radio label="有效"></el-radio>
            <el-radio label="无效"></el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="beforCloses('ruleForm')">取 消</el-button>
        <el-button type="primary" @click="submitData('ruleForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 选取终端的弹窗 -->
    <el-dialog
      v-model="dialogVisible2"
      title="选取终端"
      size="tiny"
      class="changeZhong"
      :before-close="beforClose"
    >
      <div style="width: 100%">
        <div style="width: 49%; float: left">
          <el-col :span="12" class="el_col_width">
            <div>
              <el-input
                v-model="search"
                placeholder="请输入终端名称搜索"
                @keyup.native.enter="searchOperator"
                class="order-input"
                :search="search"
              ></el-input>
              <el-button type="primary" @click="searchOperator">搜索</el-button>
            </div>
          </el-col>
          <el-col :span="12">
            <span>标签</span>
            <el-select
              v-model="searchZh"
              filterable
              placeholder="请选择"
              class="pleasechange"
            >
              <el-option
                v-for="item in searchZhData"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-col>

          <div class="interval">
            <!-- @row-click="allList" -->
            <!-- <el-checkbox
              v-model="allCheck"
              @change="allCheckEvent"
              class="changeAll"
              >全选筛选数据</el-checkbox
            > -->
            <el-table
              :data="operatorList"
              border
              @select-all="handleSelectionChange"
              @select="handleSelectionChange"
              @selection-change="handleSelectionChange"
              ref="multipleTable"
            >
              <el-table-column type="selection" width="55"> </el-table-column>
              <el-table-column label="策略名称" prop="hostId">
              </el-table-column>
              <el-table-column label="更新后版本" prop="name"></el-table-column>
            </el-table>
            <div class="table-pagination">
              <el-pagination
                :current-page="currentPage"
                :page-sizes="[10, 20, 30, 40, 50, 100, 500, 1000]"
                :page-size="limit"
                layout="prev, pager, next, jumper"
                :total="total"
                @current-change="handleCurrentPageChanged"
                @size-change="handlePageSizeChanged"
              >
              </el-pagination>
            </div>
          </div>
        </div>
        <div style="width: 49%; float: right" class="addZh">
          <p class="already">已选取终端({{ select }})</p>
          <!-- @row-click="allList" -->
          <el-table :data="terminalData" border>
            <el-table-column label="策略名称" prop="hostId"> </el-table-column>
            <el-table-column label="更新后版本" prop="name"></el-table-column>
            <el-table-column label="操作">
              <template scope="scope">
                <el-button
                  type="text"
                  size="small"
                  @click="deleteSelectItem(scope.row, scope.$index)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
          <!-- <ul>
            <li v-for="(item, index) in terminalData" v-bind:key="item.id">
              <span class="left-content">{{ item.name }}</span>
              <span
                class="right-content"
                @click="deleteSelectItem(item, index)"
              >
                <i class="el-icon-close"></i>
              </span>
            </li>
          </ul> -->
          <span slot="footer" class="dialog-footer" style="float: right">
            <el-button @click="remove">取 消</el-button>
            <el-button type="primary" @click="submitDatas('ruleForm')"
              >确 定</el-button
            >
          </span>
        </div>
      </div>
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
      title: "添加更新策略", //标题
      dialogVisible: false, //添加终端隐藏
      dialogVisible2: false, //选择终端
      form: {
        //添加更新策略的数据
        upgradePatchName: "", //策略名称
        businessPlatform: "", //更新后版本[选择器]
        businessPlatform2: "", //更新终端类型
        businessPlatform3: "", //更新数量
        description: "", //备注
        resource: "无效", //有无效
      },
      //表单规则
      rules: {
        businessPlatform: [
          { required: true, message: "请选择更新名称", trigger: "blur" },
        ],
        businessPlatform2: [
          { required: true, message: "请更新终端名称", trigger: "blur" },
        ],
        businessPlatform3: [
          { required: true, message: "请选择更新数量", trigger: "blur" },
        ],
        upgradePatchName: [
          {
            required: true,
            message: "策略名称不能为空",
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
            message: "终端不能为空",
            trigger: "blur",
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
      UpdatedVersionData: [
        {
          value: "选项1",
          label: "便民大屏",
        },
        {
          value: "选项2",
          label: "小白盒",
        },
      ],
      updateNumData: [
        {
          value: "全部终端",
          label: "全部终端",
        },
        {
          value: "部分终端",
          label: "部分终端",
        },
      ],
      fileList: [], //导入apk
      searchZh: "", //搜索终端
      searchZhData: [
        {
          value: "全部终端",
          label: "全部终端",
        },
        {
          value: "部分终端",
          label: "部分终端",
        },
      ],
      terminalData: [], //筛选选中的数据
      select: 0, //查看有多少个
      allCheck: "", //全选筛选
    };
  },
  created() {
    // let rowData = localStorage.getItem("rowData");
    // if (rowData !== "") {
    //   this.terminalData = JSON.parse(rowData);
    // }
  },
  methods: {
    //点击关闭按钮
    beforCloses(fromName) {
      this.form = {
        upgradePatchName: "", //策略名称
        businessPlatform: "", //更新后版本[选择器]
        businessPlatform2: "", //更新终端类型
        businessPlatform3: "", //更新数量
        description: "", //备注
        resource: "无效", //有无效
      };
      this.dialogVisible = false;
      if (fromName) {
        this.$nextTick(() => {
          this.$refs[fromName].resetFields(); // this.$refs.form.resetFields();
        });
      }
    },
    //选择终端 点击取消
    remove() {
      this.dialogVisible2 = false;
      localStorage.removeItem("rowData");
      this.terminalData = [];
    },
    //选择终端 关闭的回调
    beforClose(done) {
      // console.log(done, "done");
      localStorage.removeItem("rowData");
      this.terminalData = [];
      done();
    },
    //分页全选按钮
    // 分页全选-选中框改变事件
    // handleSelectionChange(val) {
    //   // 数据去重
    //   this.terminalData = this.reduceSame(val);
    //   // 选中所有选择框时勾选全选按钮
    //   if (this.terminalData.length == this.total) {
    //     this.allCheck = true;
    //   }
    //   // 将row是对象数组数据转换为字符串
    //   this.terminalData = this.terminalData
    //     .map(function (val) {
    //       return val.id;
    //     })
    //     .toString();
    //   // 选中后的数据
    //   console.log(this.terminalData);
    // },
    // 分页全选-全选按钮change事件
    // allCheckEvent() {
    //   if (this.allCheck) {
    //     // 全选选中时当前页所有数据选中
    //     this.operatorList.forEach((row) => {
    //       if (row) {
    //         this.$refs.multipleTable.toggleRowSelection(row, true);
    //       }
    //     });
    //   } else {
    //     this.$refs.multipleTable.clearSelection();
    //   }
    // },
    // 分页全选-全选时禁用选择框
    // checkSelectable(row, index) {
    //   return !this.allCheck;
    // },
    // 数组对象去重
    // reduceSame: function (arr) {
    //   let obj = {};
    //   return arr.reduce(function (prev, item) {
    //     obj[item.id] ? "" : (obj[item.id] = 1 && prev.push(item));
    //     return prev;
    //   }, []);
    // },
    //1.删除左侧栏数据
    deleteSelectItem(item, index) {
      this.terminalData.splice(index, 1);
      const hotelInfo = this.operatorList.find((n) => n.id === item.id);
      if (hotelInfo) {
        this.$refs.multipleTable.toggleRowSelection(hotelInfo, false);
      }
    },
    //2.设置行选择,每300毫秒调用一次,保存修改状态
    toggleSelection() {
      const rows = [];
      this.operatorList.forEach((item) => {
        const hotelInfo = this.terminalData.find((n) => n.id === item.id);
        if (hotelInfo) {
          rows.push(item);
        }
      });
      if (rows && rows.length > 0) {
        rows.forEach((row) => {
          this.$refs.multipleTable.toggleRowSelection(row, true);
        });
      } else if (!rows && rows.length < 0) {
        this.$refs.multipleTable.clearSelection();
      }
    },
    // 3.当表格选择项有变化时
    handleSelectionChange(selections = []) {
      for (const item of this.operatorList) {
        const listIndex = this.terminalData.findIndex(
          (n) => n.hostId === item.hostId
        );
        const selectIndex = selections.findIndex(
          (n) => n.hostId === item.hostId
        );
        if (listIndex > -1 && selectIndex <= -1) {
          this.terminalData.splice(listIndex, 1); // table没有，右侧栏有，就delete该对象
        } else if (listIndex <= -1 && selectIndex > -1) {
          this.terminalData.push(selections[selectIndex]); // table有，右侧栏没有，就add该对象
        }
      }
    },
    //点击将数据传到缓存
    // allList(row) {
    //   console.log(row, "daaaaaaaaaaa");
    //   //  let rowId =
    //   let rowData = localStorage.getItem("rowData");
    //   if (rowData !== "") {
    //     var myCart = JSON.parse(rowData);
    //   } else {
    //     var myCart = [];
    //   }
    //   var data = []; //存放将要放到缓存的数据
    //   var type = true; //控制追加和不动
    //   if (myCart !== null && myCart.length) {
    //     //如果有缓存
    //     myCart.map((res) => {
    //       if (res.id == row.id) {
    //         type = false; //表示已经有了不需要添加
    //       }
    //       data.push(res);
    //       return null;
    //     });
    //   }
    //   if (type) {
    //     data.push(row);
    //   }
    //   localStorage.setItem("rowData", JSON.stringify(data));
    //   let rowDatas = localStorage.getItem("rowData");
    //   this.select = JSON.parse(rowDatas).length;
    //   this.terminalData = JSON.parse(rowDatas);
    // },

    //终端数量的选择器
    changenum(val) {
      // console.log("部分终端");
      if (val == "部分终端") {
        this.dialogVisible2 = true;
      }
    },
    //导入终端的
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
      if (this.title === "添加更新策略") {
        //alert("添加更新策略");
      } else {
        //alert("编辑更新策略");
      }
    },
    //选择终端
    async submitDatas(ruleForm) {
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
      this.$confirm("此操作将删除该升级策略, 是否继续?", "提示", {
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
        .catch(() => {});
    },
    //创建策略
    addTerminal() {
      this.title = "添加更新策略";
      this.dialogVisible = true;
    },
    //编辑
    manageOperator(row) {
      console.log(row, "你是啥");
      this.title = "编辑更新策略";
      this.dialogVisible = true;
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
          setTimeout(() => {
            this.toggleSelection();
          }, 300);
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
    $route(to, from) {
      // 做一些路由变化的响应
      // this.loading = true;
      // this.fetchLink();
      console.log(to, from, "1111111");
    },
    // operatorList: {
    //   handler(value) {
    //     if (this.allCheck) {
    //       this.operatorList.forEach((row) => {
    //         if (row) {
    //           this.$refs.multipleTable.toggleRowSelection(row, true);
    //         }
    //       });
    //     }
    //   },
    //   deep: true,
    // },
    // "form.businessPlatform3": {
    //   handler(newName, oldName) {
    //     console.log(newName, oldName, "1252522222");
    //     if (newName == "部分终端") {
    //       this.dialogVisible2 = true;
    //     }
    //   },
    //   deep: true,
    //   immediate: true,
    // },
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
.interval {
  margin: 55px 0;
}
.already {
  margin: 30px 0 26px;
  font-size: 20px;
}
.changeAll {
  margin: 0px 0 10px;
}
</style>
<style lang="less">
.phonenum > .el-form-item__label {
  white-space: nowrap;
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
.changeZhong .el-dialog--tiny {
  width: 40%;
}
.el_col_width .order-input {
  width: 113px;
}
.pleasechange .el-input__inner {
  width: 140px;
}
.addZh {
  .el-table {
    min-height: 440px;
    height: 440px;
    overflow: auto;
  }
  .dialog-footer {
    margin: 20px 0;
  }
}
</style>
