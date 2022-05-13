<template>
  <div>
    <el-form ref="form" label-width="80px" style="margin: 10px 0">
      <el-form-item label="SPU名称">
        <el-input placeholder="SPU名称" v-model="spudata.spuName"></el-input>
      </el-form-item>
      <el-form-item label="品牌">
        <el-select placeholder="请选择品牌" v-model="spudata.tmId">
          <el-option
            :label="tm.tmName"
            v-for="tm in spuTmData"
            :key="tm.id"
            :value="tm.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <!-- textarea -->
      <el-form-item label="SPU描述">
        <el-input
          type="textarea"
          rows="4"
          v-model="spudata.description"
        ></el-input>
      </el-form-item>
      <!-- 上传图片 -->
      <el-form-item label="SPU图片">
        <el-upload
          :action="imgUploadUrl"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :file-list="spuImgList"
          :on-success="handleSuccess"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <!-- 销售属性下拉框与表格 -->
      <el-form-item label="销售属性">
        <el-select
          :placeholder="`还有${unSelectSaleAttr.length}个未选中`"
          v-model="attrInfo"
        >
          <el-option
            :label="item.name"
            :value="`${item.id}:${item.name}`"
            v-for="item in unSelectSaleAttr"
            :key="item.id"
          ></el-option>
        </el-select>
        <el-button
          type="primary"
          icon="el-icon-plus"
          style="margin-left: 10px"
          @click="addSaleAttr"
          >添加销售属性</el-button
        >
        <!-- 销售属性表格 -->
        <el-table
          style="width: 100%; margin: 10px 0"
          border
          :data="spudata.spuSaleAttrList"
        >
          <el-table-column
            prop="prop"
            label="序号"
            type="index"
            width="80"
            align="center"
          >
          </el-table-column>
          <el-table-column
            prop="saleAttrName"
            label="属性名"
            width="width"
            align="center"
          >
          </el-table-column>
          <el-table-column
            prop="prop"
            label="属性值名称列表"
            width="width"
            align="center"
          >
            <template slot-scope="{ row }">
              <el-tag
                :key="tag.id"
                v-for="(tag, index) in row.spuSaleAttrValueList"
                closable
                :disable-transitions="false"
                @close="row.spuSaleAttrValueList.splice(index, 1)"
              >
                {{ tag.saleAttrValueName }}
              </el-tag>
              <!-- @keyup.enter.native="handleInputConfirm"
                 -->
              <el-input
                class="input-new-tag"
                v-if="row.inputVisible"
                v-model="row.inputValue"
                ref="saveTagInput"
                size="mini"
                @blur="handleInputConfirm(row)"
              >
              </el-input>
              <!-- @click="showInput" -->
              <el-button
                v-else
                class="button-new-tag"
                size="mini"
                @click="addValueName(row)"
                >添加</el-button
              >
            </template>
          </el-table-column>
          <el-table-column
            prop="prop"
            label="操作"
            width="width"
            align="center"
          >
            <template slot-scope="{ $index }">
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="spudata.spuSaleAttrList.splice($index, 1)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <!-- 保存取消按钮 -->
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateSpu">保存</el-button>
        <el-button @click="cancel"
          >取消</el-button
        >
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { AppMain } from '@/layout/components';
export default {
  name: "spuForm",
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      //spu数据
      spudata: {
        //三级分类的id
        category3Id: 0,
        //描述
        description: "",
        //spu名称
        spuName: "",
        //平台的id
        tmId: "",
        //收集SPU图片的信息
        spuImageList: [
          {
            id: 0,
            imgName: "string",
            imgUrl: "string",
            spuId: 0,
          },
        ],
        //平台属性与属性值收集
        spuSaleAttrList: [
          // {
          //   // baseSaleAttrId: 0,
          //   // id: 0,
          //   // saleAttrName: "",
          //   // spuId: 0,
          //   // spuSaleAttrValueList: [
          //   //   // {
          //   //   //   baseSaleAttrId: 0,
          //   //   //   id: 0,
          //   //   //   isChecked: "string",
          //   //   //   saleAttrName: "string",
          //   //   //   saleAttrValueName: "string",
          //   //   //   spuId: 0,
          //   //   // },
          //   // ],
          // },
        ],
      },
      // spu品牌的数据
      spuTmData: [],
      // spu图标
      spuImgList: [],
      //平台销售属性
      spuSaleData: [],
      attrInfo: "",
    };
  },
  methods: {
    // 照片墙删除后的回调
    handleRemove(file, fileList) {
      this.spuImgList = fileList;
    },
    // 照片墙预览
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    // 照片墙成功的回调
    handleSuccess(response, file, fileList) {
      this.spuImgList = fileList;
    },
    async initSpuForm(spuData) {
      //获取SPU数据
      let spudata = await this.$API.spu.reqSpu(spuData.id);
      if (spudata.code == 200) {
        this.spudata = spudata.data;
      }
      //获取品牌的数据
      let spuTmData = await this.$API.spu.reqTradeMarkList();
      if (spuTmData.code == 200) {
        this.spuTmData = spuTmData.data;
      }
      //获取SPU图标
      let spuImgList = await this.$API.spu.reqSpuImageList(spuData.id);
      if (spuImgList.code == 200) {
        // 为显示照片墙需要在spuImgList中增加name url属性
        let imgArr = spuImgList.data;
        imgArr.forEach((item) => {
          item.name = item.imgName;
          item.url = item.imgUrl;
        });
        this.spuImgList = imgArr;
      }
      // 获取销售属性
      let spuSaleData = await this.$API.spu.reqBaseSaleAttrList();
      if (spuSaleData.code == 200) {
        this.spuSaleData = spuSaleData.data;
      }
    },
    // 添加销售属性
    addSaleAttr() {
      const [baseSaleAttrId, saleAttrName] = this.attrInfo.split(":");
      let newSaleAttr = {
        baseSaleAttrId,
        saleAttrName,
        spuSaleAttrValueList: [],
      };
      this.spudata.spuSaleAttrList.push(newSaleAttr);
      this.attrInfo = "";
    },
    // 销售属性列表显示Input输入框
    addValueName(row) {
      this.$set(row, "inputVisible", true);
      this.$nextTick(() => { 
        this.$refs.saveTagInput.focus()
       })
    },
    // 销售属性列表 tag blur的回调
    handleInputConfirm(row) {
      const { baseSaleAttrId, inputValue } = row;
      if (inputValue.trim() == "") {
        this.$message("输入内容不能为空");
        return;
      }
      let result = row.spuSaleAttrValueList.some(
        (item) => item.saleAttrValueName == inputValue
      );
      if (result) {
        this.$message("请勿输入相同内容");
        return;
      }
      let newSaleAttrValue = { baseSaleAttrId, saleAttrValueName: inputValue };
      row.spuSaleAttrValueList.push(newSaleAttrValue);
      row.inputVisible = false;
      row.inputValue =''
    },
    // 点击保存按钮
    async addOrUpdateSpu() {
      this.spudata.spuImageList = this.spuImgList.map((item) => {
        return {
          imageName: item.name,
          imageUrl: (item.response && item.response.data) || item.url,
        };
      });
      let result = await this.$API.spu.reqAddOrUpdateSpu(this.spudata);
      if (result.code == 200) {
        this.$emit("changeScenesToRecordsList", {scenes:'recordsList',flag:this.spudata.id?'修改':'添加'});
        this.$message({ type: "success", message: "保存成功" });
         Object.assign(this._data,this.$options.data())
      }
    },
    // 点击添加spu按钮后的回调
    async addSpuData(category3Id) {
      this.spudata.category3Id = category3Id
      let spuTmData = await this.$API.spu.reqTradeMarkList();
      if (spuTmData.code == 200) {
        this.spuTmData = spuTmData.data;
      }
      let spuSaleData = await this.$API.spu.reqBaseSaleAttrList();
      if (spuSaleData.code == 200) {
        this.spuSaleData = spuSaleData.data;
      }
    },
    // 点击取消按钮
    cancel(){
      this.$emit('changeScenesToRecordsList', {scenes:'recordsList',flag:this.spudata.id?'修改':'添加'})
      console.log(this._data,this.$options.data());
      Object.assign(this._data,this.$options.data())
    }
  },
  computed: {
    // 上传图片的url
    imgUploadUrl() {
      return process.env.VUE_APP_BASE_API + "/admin/product/fileUpload";
    },
    unSelectSaleAttr() {
      let result = this.spuSaleData.filter((item) => {
        return this.spudata.spuSaleAttrList.every((item1) => {
          return item1.saleAttrName != item.name;
        });
      });
      return result;
    },
  },
};
</script>

<style>
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 60px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>