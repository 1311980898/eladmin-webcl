<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">商品名称</label>
        <el-input v-model="query.foodName" clearable placeholder="商品名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">商品类型</label>
        <el-input v-model="query.foodType" clearable placeholder="商品类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
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
          <el-form-item label="商品名称" prop="foodName">
            <el-input v-model="form.foodName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="商品图：" prop="photoPath">
            <el-button type="primary" @click="uploadPicture">上传图片</el-button>
            <div v-if="form.photoPath" class="picture">
              <a :href="form.photoPath" target="_blank">
                <img :src="form.photoPath" class="uploaded-image">
              </a>
              <el-button class="del" type="danger" @click="deletePicture">删除</el-button>
            </div>
          </el-form-item>
          <el-form-item label="商品类型" prop="foodType">
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
        <el-table-column prop="foodName" label="商品名称" />
        <!--
        <el-table-column prop="description" label="描述" />
-->
        <el-table-column prop="imgUrl" label="商品图片" width="150">
          <template slot-scope="scope">
            <el-image
              style="width: 100px; height: 100px"
              fit="contain"
              :src="scope.row.photoPath"
              :preview-src-list="[scope.row.photoPath]"
            />
          </template>
        </el-table-column>
        <el-table-column prop="foodType" label="商品类型">
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
import axios from 'axios'

const defaultForm = { foodId: null, foodName: null, description: null, foodType: null, price: null, supplier: null, productionDate: null, expiryDate: null, storageConditions: null, unit: null, stockQuantity: null, photoPath: null, createdAt: null, updatedAt: null }
export default {
  name: 'CmsFood',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['food_type', 'food_supplier', 'unit_type'],
  cruds() {
    return CRUD({ title: '商品', url: 'api/cmsFood', idField: 'foodId', sort: 'foodId,desc', crudMethod: { ...crudCmsFood }})
  },
  data() {
    return {
      rules: {
        foodName: [
          { required: true, message: '商品名称不能为空', trigger: 'blur' }
        ],
        foodType: [
          { required: true, message: '商品类型不能为空', trigger: 'blur' }
        ],
        price: [
          { required: true, message: '价格不能为空', trigger: 'blur' }
        ],
        supplier: [
          { required: false, message: '供应商不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'foodName', display_name: '商品名称' },
        { key: 'foodType', display_name: '商品类型' },
        { key: 'price', display_name: '价格' },
        { key: 'supplier', display_name: '供应商' },
        { key: 'storageConditions', display_name: '存储条件' },
        { key: 'unit', display_name: '单位' },
        { key: 'stockQuantity', display_name: '库存数量' }
      ]
    }
  },
  computed: {
    handlePicture() {
      const list = this.form.photoPath
      const arr = []
      for (let i = 0; i < list.length; i++) {
        const item = list[i]
        if (typeof item === 'string') arr.push(item)
        else arr.push(URL.createObjectURL(item))
      }
      return arr
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    },
    // 上传图片
    uploadPicture() {
      var input = document.createElement('input')
      input.type = 'file'
      input.accept = '.gif,.jpg,.jpeg,.bmp,.png,.GIF,.JPG,.PNG,.BMP'
      input.onchange = (e) => {
        const file = e.target.files[0]
        const formData = new FormData()
        formData.append('file', file)

        // 发送图片上传请求
        axios.post('http://localhost:8000/api/localStorage/picturesV2', formData)
          .then(response => {
            this.form.photoPath = response.data.path // 假设后端返回的字段名为imageUrl
          })
          .catch(error => {
            console.error('上传失败:', error)
          })
      }
      input.click()
    },
    // 删除图片
    deletePicture() {
      this.form.photoPath = null
    }
  }
}
</script>

<style scoped>
.uploaded-image {
  max-width: 200px; /* 设置图片的最大宽度 */
  max-height: 200px; /* 设置图片的最大高度 */
}

</style>

