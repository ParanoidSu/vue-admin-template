<template>
  <div>
    <el-card style="margin: 20px 0">
      <CategorySelect
        @getCategoryId="getCategoryId"
        :show="scenes == 'recordsList'"
      ></CategorySelect>
    </el-card>
    <el-card>
      <div v-show="scenes == 'recordsList'">
        <el-button
          type="primary"
          icon="el-icon-plus"
          @click="addSpu"
          :disabled="!categoryId.category3Id"
          >添加SPU</el-button
        >
        <!-- 数据列表 -->
        <el-table style="width: 100%; margin: 20px 0" border :data="records">
          <el-table-column
            prop="prop"
            type="index"
            label="序号"
            align="center"
            width="80"
          >
          </el-table-column>
          <el-table-column
            prop="spuName"
            label="Spu名称"
            align="center"
            width="width"
          >
          </el-table-column>
          <el-table-column
            prop="description"
            label="Spu描述"
            align="center"
            width="width"
          >
          </el-table-column>
          <el-table-column
            prop="prop"
            label="操作"
            align="center"
            width="width"
          >
            <template slot-scope="{ row }">
              <!-- 操作按钮 -->
              <el-button
                type="success"
                icon="el-icon-plus"
                size="mini"
                title="新增"
                @click="addSkuForm(row)"
              ></el-button>
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                title="编辑"
                @click="editSpuForm(row)"
              ></el-button>
              <el-button
                type="info"
                icon="el-icon-info"
                size="mini"
                title="信息"
                @click="showDialog(row)"
              ></el-button>
              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
                style="margin: 0px 10px"
              >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除"
                  slot="reference"
                ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>

        <!-- dialog -->
      
        <el-dialog :title="`${spu.spuName}的列表`" :visible.sync="dialogTableVisible" @before-close='beforCloseDialog'>
          <el-table
            :data="skuList"
             v-loading="loading"
            style="width: 100%">
            <el-table-column
              prop="skuName"
              label="名字"
              width="width">
            </el-table-column>
            <el-table-column
              prop="price"
              label="价格"
              width="width">
            </el-table-column>
            <el-table-column
              prop="weight"
              label="重量"
              width="width">
            </el-table-column>
            <el-table-column
              prop="默认图片"
              label="默认图片"
              width="width">
              <template slot-scope={row}>
               <img :src="row.skuDefaultImg" alt="" style="width:100px;height:100px">
              </template>
            </el-table-column>
          </el-table>  
        </el-dialog>
        

        <!-- 分页器 -->
        <el-pagination
          style="text-align: center"
          :current-page="page"
          :page-sizes="[3, 5, 7]"
          :page-size="limit"
          layout="  prev, pager, next, jumper,->,sizes,total"
          :total="total"
          @current-change="handleCurrentChange"
          @size-change="handleSizeChange"
        >
        </el-pagination>
      </div>
      <!-- 数据表格 -->

      <skuForm
        v-show="scenes == 'skuForm'"
        ref="Sku"
        @skuFormCancel="skuFormCancel"
      ></skuForm>
      <spuForm
        v-show="scenes == 'spuForm'"
        @changeScenesToRecordsList="changeScenesToRecordsList"
        ref="Spu"
      ></spuForm>
    </el-card>
  </div>
</template>

<script>
import skuForm from "@/views/product/Spu/skuForm";
import spuForm from "@/views/product/Spu/spuForm";
export default {
  name: "Spu",
  data() {
    return {
      categoryId: {},
      //分页器当前页数
      page: 1,
      //分页器单页显示个数
      limit: 3,
      //spu列表数据
      records: [],
      //分页器总数据
      total: null,
      //recordsList:数据表格   spuForm   skuForm
      scenes: "recordsList",
      // 控制dialog显示或隐藏
      dialogTableVisible:false,
      spu:{},
      // 存储sku列表数据
      skuList:[],
      loading:true
    };
  },

  components: {
    skuForm,
    spuForm,
  },
  methods: {
    //获取categoryid
    getCategoryId(categoryId) {
      this.categoryId = { ...categoryId };
      this.getSquList();
    },
    //发请求渲染列表
    async getSquList(pages = 1) {
      this.page = pages;
      const { page, limit } = this;
      const { category3Id } = this.categoryId;
      let result = await this.$API.spu.reqSquList(page, limit, category3Id);
      if (result.code == 200) {
        this.records = result.data.records;
        this.total = result.data.total;
      }
    },
    // 分页器page改变重新渲染列表
    handleCurrentChange(page) {
      this.page = page;
      this.getSquList(this.page);
    },
    // 分页器size改变重新渲染列表
    handleSizeChange(limit) {
      this.limit = limit;
      this.getSquList();
    },
    // 点击添加spu按钮
    addSpu() {
      this.scenes = "spuForm";
      const { category3Id } = this.categoryId;
      this.$refs.Spu.addSpuData(category3Id);
    },
    // 点击编辑按钮
    editSpuForm(row) {
      this.scenes = "spuForm";
      // 传递row的值给子组件的initSpuForm
      this.$refs.Spu.initSpuForm(row);
    },
    // 点击新增按钮
    addSkuForm(row) {
      const { category1Id, category2Id } = this.categoryId;
      this.scenes = "skuForm";
      this.$refs.Sku.getDate(category1Id, category2Id, row);
    },
    // spuForm点击取消按钮后的回调
    changeScenesToRecordsList({ scenes, flag }) {
      this.scenes = scenes;
      if (flag == "修改") {
        this.getSquList(this.page);
      } else {
        this.page = 1;
        this.getSquList();
      }
    },
    // 删除spu
    async deleteSpu(row) {
      let result = await this.$API.spu.reqDeleteSpu(row.id);
      if (result.code == 200) {
        this.$message({ type: "success", message: "删除成功" });
        this.getSquList(this.records.length > 1 ? this.page : this.page - 1);
      }
    },
    skuFormCancel(scenes) {
      this.scenes = scenes;
    },
    async showDialog(spu){
      this.dialogTableVisible = true
      this.spu = spu
      let result = await this.$API.spu.reqSkuList(spu.id)
      if (result.code ==200) {
        this.skuList = result.data
        this.loading = false
      }
    },
    beforCloseDialog(done){
      this.loading = true
      this.skuList = []
      done()
    }
  },
};
</script>

<style>
</style>