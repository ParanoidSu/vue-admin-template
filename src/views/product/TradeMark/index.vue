<template>
  <div>
    <!-- 添加按钮 -->
    <el-button
      type="primary"
      icon="el-icon-plus"
      style="margin: 10px 0"
      @click="showDialog"
      >添加</el-button
    >
    <!-- 主体表格 -->
    <el-table style="width: 100%" border :data="list">
      <!-- 序号列 -->
      <el-table-column
        prop="prop"
        label="序号"
        width="80px"
        align="center"
        type="index"
      >
      </el-table-column>
      <!-- 品牌名称列 -->
      <el-table-column
        prop="tmName"
        label="品牌名称"
        width="width"
        align="center"
      >
      </el-table-column>
      <!-- 品牌LOGO列 -->
      <el-table-column
        prop="logoUrl"
        label="品牌Logo"
        width="width"
        align="center"
      >
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 100px" />
        </template>
      </el-table-column>
      <!-- 操作按钮列 -->
      <el-table-column prop="prop" label="操作" width="width" align="center">
        <template slot-scope="{ row }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="updateTradeMark(row)"
            >修改</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteTradeMark(row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- dialog对话框 -->
    <el-dialog
      :title="tmForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <el-form style="width: 80%" :model="tmForm" ref="ruleform" :rules="rules">
        <el-form-item label="品牌名称" label-width="100px" prop="tmName">
          <el-input autocomplete="off" v-model="tmForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌LOGO" label-width="100px" prop="logoUrl">
          <!-- 上传图片 -->
          <!-- :show-file-list="false"
                图片上传成功
                :on-success="handleAvatarSuccess"
                图片上传之前
                :before-upload="beforeAvatarUpload" -->
          <el-upload
            class="avatar-uploader"
            :action="imgUploadUrl"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件,且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark('ruleform')"
          >确 定</el-button
        >
      </div>
    </el-dialog>
    <!-- 上传图片 -->

    <!-- 分页器 -->
    <!-- 
       layout:布局
       current-page:当前的第几页
       total:一共展示多少条数据
       page-size:一页展示多少条数据
       page-sizes:可以设置一页展示多少条数据的值
       pager-count:页码按钮的数量 只能为奇数
       current-change：点击时获取更新当前页码并再发一遍请求
       size-change：点击时获取更新limit并再发请求
      -->
    <el-pagination
      style="margin-top: 20px; text-align: center"
      layout="prev, pager, next, jumper, -> , total,sizes"
      :current-page="page"
      :total="total"
      :page-size="limit"
      :page-sizes="[3, 5, 10]"
      :pager-count="5"
      @current-change="getPageList"
      @size-change="handleSizeChange"
    >
    </el-pagination>
  </div>
</template>

<script>
export default {
  name: "TradeMark",
  data() {
    // 表单自定义验证
    var validatetmName = (rule, value, callback) => {
      if (value.trim().length < 2 || value.trim().length > 10) {
        callback(new Error("品牌名应为2-10位数"));
      } else {
        callback;
      }
    };
    return {
      //当前页面
      page: 1,
      //每页显示数据个数
      limit: 3,
      //数据总数
      total: 0,
      //数据存储
      list: [],
      //对话框显示
      dialogFormVisible: false,
      //上传图片使用
      imageUrl: "",
      tmForm: {
        tmName: "",
        logoUrl: "",
      },
      rules: {
        //品牌名称验证
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          // {
          //   min: 2,
          //   max: 10,
          //   message: "长度在 2 到 10 个字符",
          //   trigger: "blur",
          // },
          //自定义验证写法
          { validator: validatetmName, trigger: "blur" },
        ],
        //品牌logo验证
        logoUrl: [{ required: true, message: "请上传图片" }],
      },
    };
  },
  methods: {
    async getPageList(pager = 1) {
      //获取点击的页面数，默认值是1
      this.page = pager;
      //解构
      const { page, limit } = this;
      let result = await this.$API.tradeMark.reqTradeMarkList(page, limit);
      if (result.code == 200) {
        this.total = result.data.total;
        this.list = result.data.records;
      }
    },
    handleSizeChange(limit) {
      this.limit = limit;
      this.getPageList();
    },
    //点击添加按钮时显示dialog
    showDialog() {
      //点击添加按钮时显示dialog
      this.dialogFormVisible = true;
      this.tmForm = { tmName: "", logoUrl: "" };
    },
    //点击修改按钮时显示dialog并更新数据
    updateTradeMark(row) {
      this.dialogFormVisible = true;
      this.tmForm = { ...row };
    },
    //上传图片成功
    handleAvatarSuccess(res, file) {
      this.tmForm.logoUrl = res.data;
      console.log(res, file);
    },
    // 上传图片之前图片类型与大小
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    // 点击添加或者修改品牌dialog
    addOrUpdateTradeMark(formName) {
      //  验证表单规则
      this.$refs[formName].validate(async (valid) => {
        if (valid) {
          // 成功执行
          this.dialogFormVisible = false;
          let result = await this.$API.tradeMark.reqAddOrUpdateTradeMark(
            this.tmForm
          );
          if (result.code == 200) {
            this.$message({
              type: "success",
              message: this.tmForm.id ? "修改品牌成功" : "添加品牌成功",
            });
            this.getPageList(this.tmForm.id ? this.page : 1);
          }
        } else {
          // 失败执行
          return false;
        }
      });
    },
    deleteTradeMark(row) {
      this.$confirm(
        `你确定删除${row.tmName}?`,
        "提示",
        {
          confirmButtonText: "确认",
          cancelButtonText: "取消",
          type:'warning'
        }
      )
        .then(async () => {
          let result = await this.$API.tradeMark.reqDeleteTradeMark(row.id);
          console.log(result);
          if (result.code ==200) {
            this.$message({
             type: "success",
             message: "删除成功",
          });
           this.getPageList(this.list.length >1?this.page :this.page -1)
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message:'已取消删除'
          });
        });
    },
  },
  mounted() {
    //加载时发请求
    this.getPageList();
  },
  computed: {
    // 上传图片的url
    imgUploadUrl() {
      return process.env.VUE_APP_BASE_API + "/admin/product/fileUpload";
    },
  },
};
</script>

<style>
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
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>