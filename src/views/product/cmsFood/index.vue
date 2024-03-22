<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">食品名称</label>
        <el-input v-model="query.foodName" clearable placeholder="食品名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">食品类型</label>
        <el-input v-model="query.foodType" clearable placeholder="食品类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">供应商</label>
        <el-input v-model="query.supplier" clearable placeholder="供应商" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">存储条件</label>
        <el-input v-model="query.storageConditions" clearable placeholder="存储条件" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.productionDate"
          start-placeholder="productionDateStart"
          end-placeholder="productionDateStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.expiryDate"
          start-placeholder="expiryDateStart"
          end-placeholder="expiryDateStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.createdAt"
          start-placeholder="createdAtStart"
          end-placeholder="createdAtStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.updatedAt"
          start-placeholder="updatedAtStart"
          end-placeholder="updatedAtStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="食品名称" prop="foodName">
            <el-input v-model="form.foodName" style="width: 370px;" />
          </el-form-item>
          <el-form-item v-if="crud.status.add" label="食品图片">
            <el-upload
              ref="upload"
              :limit="1"
              :before-upload="beforeUpload"
              :auto-upload="false"
              :headers="headers"
              :on-success="handleSuccess"
              :on-error="handleError"
              :action="fileUploadApi + '?name=' + form.name"
            >
              <div class="eladmin-upload"><i class="el-icon-upload" /> 添加图片</div>
            </el-upload>
          </el-form-item>
          <el-form-item label="食品类型" prop="foodType">
            <el-radio v-for="item in dict.food_type" :key="item.id" v-model="form.foodType" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="价格" prop="price">
            <el-input v-model="form.price" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="供应商" prop="supplier">
            <el-radio v-for="item in dict.food_supplier" :key="item.id" v-model="form.supplier" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="生产日期">
            <el-date-picker v-model="form.productionDate" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="过期日期">
            <el-date-picker v-model="form.expiryDate" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="存储条件">
            <el-input v-model="form.storageConditions" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="单位">
            <el-radio v-for="item in dict.unit_type" :key="item.id" v-model="form.unit" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="库存数量">
            <el-input v-model="form.stockQuantity" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="foodId" label="食品ID" />
        <el-table-column prop="foodName" label="食品名称" />
        <!--
        <el-table-column prop="description" label="描述" />
-->
        <el-table-column prop="imgUrl" label="食品图片" width="150">
          <template slot-scope="scope">
            <el-image
              style="width: 100px; height: 100px"
              fit="contain"
              :src="scope.row.photoPath"
              :preview-src-list="[scope.row.photoPath]"
            />
          </template>
        </el-table-column>
        <el-table-column prop="foodType" label="食品类型">
          <template slot-scope="scope">
            {{ dict.label.food_type[scope.row.foodType] }}
          </template>
        </el-table-column>
        <el-table-column prop="price" label="价格" />
        <el-table-column prop="supplier" label="供应商">
          <template slot-scope="scope">
            {{ dict.label.food_supplier[scope.row.supplier] }}
          </template>
        </el-table-column>
        <el-table-column prop="expiryDate" label="过期日期" />
        <el-table-column prop="storageConditions" label="存储条件" />
        <el-table-column prop="unit" label="单位">
          <template slot-scope="scope">
            {{ dict.label.unit_type[scope.row.unit] }}
          </template>
        </el-table-column>
        <el-table-column prop="stockQuantity" label="库存数量" />
        <el-table-column prop="createdAt" label="创建时间" />
        <el-table-column v-if="checkPer(['admin','cmsFood:edit','cmsFood:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudCmsFood from '@/api/cmsFood'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { foodId: null, foodName: null, description: null, foodType: null, price: null, supplier: null, productionDate: null, expiryDate: null, storageConditions: null, unit: null, stockQuantity: null, photoPath: null, createdAt: null, updatedAt: null }
export default {
  name: 'CmsFood',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['food_type', 'food_supplier', 'unit_type'],
  cruds() {
    return CRUD({ title: '食品', url: 'api/cmsFood', idField: 'foodId', sort: 'foodId,desc', crudMethod: { ...crudCmsFood }})
  },
  data() {
    return {
      rules: {
        foodName: [
          { required: true, message: '食品名称不能为空', trigger: 'blur' }
        ],
        foodType: [
          { required: true, message: '食品类型不能为空', trigger: 'blur' }
        ],
        price: [
          { required: true, message: '价格不能为空', trigger: 'blur' }
        ],
        supplier: [
          { required: true, message: '供应商不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'foodName', display_name: '食品名称' },
        { key: 'foodType', display_name: '食品类型' },
        { key: 'price', display_name: '价格' },
        { key: 'supplier', display_name: '供应商' },
        { key: 'storageConditions', display_name: '存储条件' },
        { key: 'unit', display_name: '单位' },
        { key: 'stockQuantity', display_name: '库存数量' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    },
    // 上传文件
    upload() {
      this.$refs.upload.submit()
    },
    beforeUpload(file) {
      let isLt2M = true
      isLt2M = file.size / 1024 / 1024 < 100
      if (!isLt2M) {
        this.loading = false
        this.$message.error('上传文件大小不能超过 100MB!')
      }
      this.form.name = file.name
      return isLt2M
    },
    handleSuccess(response, file, fileList) {
      this.crud.notify('上传成功', CRUD.NOTIFICATION_TYPE.SUCCESS)
      this.$refs.upload.clearFiles()
      this.crud.status.add = CRUD.STATUS.NORMAL
      this.crud.resetForm()
      this.crud.toQuery()
    },
    // 监听上传失败
    handleError(e, file, fileList) {
      const msg = JSON.parse(e.message)
      this.$notify({
        title: msg.message,
        type: 'error',
        duration: 2500
      })
      this.loading = false
    }
  }
}
</script>

<style scoped>

</style>
