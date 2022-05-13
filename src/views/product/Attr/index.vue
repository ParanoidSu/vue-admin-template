<template>
  <div>
    <el-card style="margin: 20px 0">
      <!-- 让三级列表在添加属性时禁用 -->
      <CategorySelect @getCategoryId="getCategoryId" :show='isShowTable'></CategorySelect>
    </el-card>
    <el-card>
      <!-- 添加按钮 -->

      <el-button
        type="primary"
        icon="el-icon-plus"
        @click="addAttr"
        v-show="isShowTable"
        :disabled="!categoryId.category3Id"
        >添加属性</el-button
      >

      <!-- 数据表格 -->
      <div v-show="isShowTable">
        <el-table
          style="width: 100%; margin: 20px 0 0 0"
          border
          :data="attrList"
        >
          <el-table-column
            prop="prop"
            label="序号"
            width="80px"
            type="index"
            align="center"
          >
          </el-table-column>
          <el-table-column prop="attrName" label="属性名称" min-width="4">
          </el-table-column>
          <el-table-column prop="prop" label="属性值列表" min-width="4">
            <template slot-scope="{ row }">
              <el-tag
                type="success"
                v-for="attrValue in row.attrValueList"
                :key="attrValue.id"
                style="margin: 0 10px"
              >
                {{ attrValue.valueName }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" min-width="1">
            <template slot-scope="{ row }">
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                @click="editAttr(row)"
              ></el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <!-- 添加属性 ||修改属性 的结构 -->
      <div v-show="!isShowTable">
        <el-form :inline="true" label-width="80px">
          <el-form-item label="属性名">
            <el-input
              placeholder="请输入属性名"
              v-model="attrInfo.attrName"
            ></el-input>
          </el-form-item>
        </el-form>
        <el-button
          type="primary"
          icon="el-icon-plus"
          @click="addValueName"
          :disabled="!attrInfo.attrName"
          >添加属性值</el-button
        >
        <el-button @click="isShowTable = true">取消</el-button>
        <el-table
          style="width: 100%; margin: 20px 0"
          border
          :data="attrInfo.attrValueList"
        >
          <el-table-column type="index" align="center" label="序号" width="80">
          </el-table-column>
          <el-table-column align="center" label="属性值名称" width="width">
            <template slot-scope="{ row }">
              <el-input
                placeholder="请输入属性值名称"
                v-model.trim="row.valueName"
                size="mini"
                v-if="row.flag"
                @blur="showSpan(row)"
                ref="valueName"
                @keyup.native.enter="showSpan(row)"
              ></el-input>
              <span v-else @click="showInput(row)" style="display: block">{{
                row.valueName
              }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="操作" width="width">
            <template slot-scope="{ row,$index }">
             <!-- popconfirm气泡确认框 -->
              <el-popconfirm :title="`确认删除${row.valueName}吗？`" @onConfirm="deleteValueName($index)">
                 <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                slot="reference"
              ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-button type="primary" @click="saveAttrInfo" :disabled="attrInfo.attrValueList.length <1">保存</el-button>
        <el-button @click="isShowTable = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: "Attr",
  data() {
    return {
      categoryId: {},
      attrList: [],
      isShowTable: true,
      attrInfo: {
        attrName: "", //属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        categoryId: 0, //三级分类的id
        categoryLevel: 3, //因为服务器也需要区分几级id
      },
    };
  },
  methods: {
    //获得三级分类的id
    getCategoryId(categoryId) {
      this.categoryId = { ...categoryId };
      this.getAttrList();
    },
    //发请求接受数据渲染页面
    async getAttrList() {
      const { category1Id, category2Id, category3Id } = this.categoryId;
      let result = await this.$API.attr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      );
      if (result.code == 200) {
        this.attrList = result.data;
      }
    },
    // 添加属性值名称
    addValueName() {
      this.attrInfo.attrValueList.push({
        attrId: this.attrInfo.id,
        valueName: "",
        flag: true,
      });
      this.$nextTick(() => {
        this.$refs.valueName.focus();
      });
    },
    addAttr() {
      // 点击后不显示数据列表
      this.isShowTable = false;
      // 将attrInfo清空并获取category3Id
      this.attrInfo = {
        attrName: "",
        attrValueList: [],
        categoryId: this.categoryId.category3Id,
        categoryLevel: 3,
      };
    },
    //点击编辑按钮
    editAttr(row) {
      this.isShowTable = false;
      this.attrInfo = JSON.parse(JSON.stringify(row));
      this.attrInfo.attrValueList.forEach((item) => {
        this.$set(item, "flag", false);
      });
    },
    // input失焦调用
    showSpan(row) {
      if (row.valueName.trim() == "") {
        this.$message("输入参数不能为空");
        this.$refs.valueName.focus();
        return;
      }
      let isRepat = this.attrInfo.attrValueList.some((item) => {
        // 排除自身
        if (row != item) {
          return row.valueName == item.valueName;
        }
      });
      if (isRepat) {
        return;
      }
      //规定input不能有空格
      row.valueName = row.valueName.replace(/\s/g,"")
      row.flag = false;
    },
    // 单击span后调用
    showInput(row) {
      row.flag = true;
      this.$nextTick(() => {
        this.$refs.valueName.focus();
      });
    },
    // popconfirm气泡确认删除回调
    deleteValueName(index){
      this.attrInfo.attrValueList.splice(index,1)
    },
    //保存数据并重新渲染页面
    saveAttrInfo(){
     this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter((item) => { 
        if (item.valueName.trim() !='') {
          delete item.flag
          return true
        }
       })
      this.$API.attr.reqAddOrUpdateAttr(this.attrInfo)
      this.isShowTable = true
      this.getAttrList()
      
    }
  },
};
</script>

<style>
</style>