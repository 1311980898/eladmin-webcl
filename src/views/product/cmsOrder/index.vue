<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">订单号</label>
        <el-input v-model="query.orderNo" clearable placeholder="订单号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">商品id</label>
        <el-input v-model="query.productId" clearable placeholder="商品id" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">订单金额</label>
        <el-input v-model="query.totalAmount" clearable placeholder="订单金额" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">订单状态</label>
        <el-input v-model="query.status" clearable placeholder="订单状态" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.orderDate"
          start-placeholder="orderDateStart"
          end-placeholder="orderDateStart"
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
          <el-form-item label="id">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="订单号">
            <el-input v-model="form.orderNo" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="商品id">
            <el-input v-model="form.productId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="下单时间">
            <el-input v-model="form.orderDate" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="订单金额">
            <el-input v-model="form.totalAmount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="订单状态">
            <el-input v-model="form.status" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建时间">
            <el-input v-model="form.createdAt" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="修改时间">
            <el-input v-model="form.updatedAt" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="商品数量">
            <el-input v-model="form.orderNum" style="width: 370px;" />
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
        <el-table-column prop="id" label="id" />
        <el-table-column prop="orderNo" label="订单号" />
        <el-table-column prop="productId" label="商品id" />
        <el-table-column prop="orderDate" label="下单时间" />
        <el-table-column prop="totalAmount" label="订单金额" />
        <el-table-column prop="status" label="订单状态" />
        <el-table-column prop="createdAt" label="创建时间" />
        <el-table-column prop="updatedAt" label="修改时间" />
        <el-table-column prop="orderNum" label="商品数量" />
        <el-table-column v-if="checkPer(['admin','cmsOrder:edit','cmsOrder:del'])" label="操作" width="150px" align="center">
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
import crudCmsOrder from '@/api/cmsOrder'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, orderNo: null, productId: null, orderDate: null, totalAmount: null, status: null, createdAt: null, updatedAt: null, orderNum: null }
export default {
  name: 'CmsOrder',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: 'cmsOrder', url: 'api/cmsOrder', idField: 'id', sort: 'id,desc', crudMethod: { ...crudCmsOrder }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'cmsOrder:add'],
        edit: ['admin', 'cmsOrder:edit'],
        del: ['admin', 'cmsOrder:del']
      },
      rules: {
      },
      queryTypeOptions: [
        { key: 'orderNo', display_name: '订单号' },
        { key: 'productId', display_name: '商品id' },
        { key: 'totalAmount', display_name: '订单金额' },
        { key: 'status', display_name: '订单状态' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
