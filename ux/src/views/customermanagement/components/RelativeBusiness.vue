<template>
  <div class="rc-cont"
       v-empty="nopermission"
       xs-empty-icon="nopermission"
       xs-empty-text="暂无权限">
    <flexbox v-if="!isSeas"
             class="rc-head"
             direction="row-reverse">
      <el-button class="rc-head-item"
                 @click.native="createClick"
                 type="primary">新建商机
      </el-button>
    </flexbox>
    <el-table :data="list"
              :height="tableHeight"
              stripe
              style="width: 100%;border: 1px solid #E6E6E6;"
              :header-cell-style="headerRowStyle"
              :cell-style="cellStyle"
              @row-click="handleRowClick">
      <el-table-column v-for="(item, index) in fieldList"
                       :key="index"
                       show-overflow-tooltip
                       :prop="item.prop"
                       :formatter="fieldFormatter"
                       :label="item.label">
      </el-table-column>
    </el-table>

    <c-r-m-full-screen-detail :visible.sync="showFullDetail"
                              crmType="business"
                              :id="businessId"></c-r-m-full-screen-detail>
    <c-r-m-create-view v-if="isCreate"
                       crm-type="business"
                       :action="createActionInfo"
                       @save-success="createSaveSuccess"
                       @hiden-view="isCreate=false"></c-r-m-create-view>
  </div>
</template>

<script type="text/javascript">
import loading from '../mixins/loading'
import CRMCreateView from './CRMCreateView'
import { crmBusinessIndex } from '@/api/customermanagement/business'

export default {
  name: 'relative-business', //相关联系人商机  可能再很多地方展示 放到客户管理目录下（新建时仅和客户进行关联）
  components: {
    CRMFullScreenDetail: () => import('./CRMFullScreenDetail.vue'),
    CRMCreateView
  },
  computed: {},
  mixins: [loading],
  data() {
    return {
      nopermission: false,
      list: [],
      fieldList: [],
      tableHeight: '400px',
      showFullDetail: false,
      isCreate: false, // 控制新建
      businessId: '', // 查看全屏联系人详情的 ID
      /** 格式化规则 */
      formatterRules: {},
      // 创建的相关信息
      createActionInfo: { type: 'relative', crmType: this.crmType, data: {} }
    }
  },
  watch: {
    id: function(val) {
      this.list = []
      this.getDetail()
    }
  },
  props: {
    /** 模块ID */
    id: [String, Number],
    /** 没有值就是全部类型 有值就是当个类型 */
    crmType: {
      type: String,
      default: ''
    },
    /** 是公海 默认是客户 */
    isSeas: {
      type: Boolean,
      default: false
    },
    /** 联系人人下 新建商机 需要联系人里的客户信息  合同下需要客户和商机信息 */
    detail: {
      type: Object,
      default: () => {
        return {}
      }
    }
  },
  mounted() {
    this.getDetail()
  },
  activated: function() {},
  deactivated: function() {},
  methods: {
    getFieldList() {
      this.fieldList.push({ prop: 'name', width: '200', label: '商机名称' })
      this.fieldList.push({
        prop: 'money',
        width: '200',
        label: '商机金额'
      })
      this.fieldList.push({
        prop: 'customer_id',
        width: '200',
        label: '客户名称'
      })
      this.fieldList.push({
        prop: 'type_id',
        width: '200',
        label: '商机状态组'
      })
      this.fieldList.push({ prop: 'status_id', width: '200', label: '状态' })

      // 为客户名称 商机状态组 状态 加入字段格式化展示规则
      function crmFieldFormatter(info) {
        return info ? info.name : ''
      }
      this.formatterRules['customer_id'] = {
        type: 'crm',
        formatter: crmFieldFormatter
      }
      function fieldFormatter(info) {
        return info ? info : ''
      }
      this.formatterRules['type_id'] = {
        type: 'crm',
        formatter: fieldFormatter
      }
      this.formatterRules['status_id'] = {
        type: 'crm',
        formatter: fieldFormatter
      }
    },
    getDetail() {
      this.loading = true
      crmBusinessIndex({
        pageType: 'all',
        customer_id: this.id
      })
        .then(res => {
          if (this.fieldList.length == 0) {
            this.getFieldList()
          }
          this.nopermission = false
          this.loading = false
          this.list = res.data.list
        })
        .catch(data => {
          if (data.code == 102) {
            this.nopermission = true
          }
          this.loading = false
        })
    },
    /** 格式化字段 */
    fieldFormatter(row, column) {
      // 如果需要格式化
      var aRules = this.formatterRules[column.property]
      if (aRules) {
        if (aRules.type === 'crm') {
          if (column.property) {
            return aRules.formatter(row[column.property + '_info'])
          } else {
            return ''
          }
        } else {
          return aRules.formatter(row[column.property])
        }
      }
      return row[column.property]
    },
    //当某一行被点击时会触发该事件
    handleRowClick(row, column, event) {
      this.businessId = row.business_id
      this.showFullDetail = true
    },
    /** 通过回调控制表头style */
    headerRowStyle({ row, column, rowIndex, columnIndex }) {
      return { textAlign: 'center' }
    },
    /** 通过回调控制style */
    cellStyle({ row, column, rowIndex, columnIndex }) {
      return { textAlign: 'center' }
    },
    /** 新建 */
    createClick() {
      /** 客户 和 联系人 下可新建商机  */
      if (this.crmType == 'contacts') {
        this.createActionInfo.data['customer'] = this.detail.customer_id_info
      } else if (this.crmType == 'customer') {
        this.createActionInfo.data['customer'] = this.detail
      }
      this.isCreate = true
    },
    createSaveSuccess() {
      this.getDetail()
    }
  }
}
</script>
<style lang="scss" scoped>
@import '../styles/relativecrm.scss';
</style>
