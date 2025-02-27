<template>
  <div>
    <vxe-table
      show-overflow
      show-header-overflow
      stripe
      size="small"
      class="ops-stripe-table"
      :data="tableData"
      v-bind="ci_id ? { maxHeight: `${windowHeight - 94}px` } : { height: `${windowHeight - 225}px` }"
    >
      <vxe-column field="trigger_name" :title="$t('cmdb.history.triggerName')"> </vxe-column>
      <vxe-column field="type" :title="$t('type')">
        <template #default="{ row }">
          <span v-if="row.trigger && row.trigger.attr_id">{{ $t('cmdb.ciType.triggerDate') }}</span>
          <span v-else-if="row.trigger && !row.trigger.attr_id">{{ $t('cmdb.ciType.triggerDataChange') }}</span>
        </template>
      </vxe-column>
      <vxe-column :title="$t('cmdb.history.event')">
        <template #default="{ row }">
          <span v-if="row.operate_type === '0'">{{ $t('cmdb.ciType.addInstance') }}</span>
          <span v-else-if="row.operate_type === '1'">{{ $t('cmdb.ciType.deleteInstance') }}</span>
          <span v-else-if="row.operate_type === '2'">{{ $t('cmdb.ciType.changeInstance') }}</span>
        </template>
      </vxe-column>
      <vxe-column :title="$t('cmdb.history.action')">
        <template #default="{ row }">
          <span v-if="row.webhook">Webhook</span>
          <span v-else-if="row.notify">{{ $t('cmdb.ciType.notify') }}</span>
        </template>
      </vxe-column>
      <vxe-column :title="$t('cmdb.history.status')">
        <template #default="{ row }">
          <a-tag color="green" v-if="row.is_ok">{{ $t('cmdb.history.done') }}</a-tag>
          <a-tag color="red" v-else>{{ $t('cmdb.history.undone') }}</a-tag>
        </template>
      </vxe-column>
      <vxe-column :title="$t('cmdb.history.triggerTime')">
        <template #default="{row}">
          {{ row.updated_at || row.created_at }}
        </template>
      </vxe-column>
    </vxe-table>
    <div :style="{ textAlign: 'right' }" v-if="!ci_id">
      <a-pagination
        size="small"
        show-size-changer
        show-quick-jumper
        :page-size-options="pageSizeOptions"
        :current="tablePage.currentPage"
        :total="tablePage.totalResult"
        :show-total="(total, range) => $t('cmdb.history.totalItems', { total: total })"
        :page-size="tablePage.pageSize"
        :default-current="1"
        @change="pageOrSizeChange"
        @showSizeChange="pageOrSizeChange"
      >
      </a-pagination>
    </div>
  </div>
</template>

<script>
import { getCiTriggers, getCiTriggersByCiId } from '../../../api/history'
export default {
  name: 'TriggerTable',
  props: {
    ci_id: {
      type: Number,
      default: null,
    },
  },
  data() {
    return {
      tableData: [],
      tablePage: {
        currentPage: 1,
        pageSize: 50,
        totalResult: 0,
      },
      pageSizeOptions: ['50', '100', '200'],
    }
  },
  computed: {
    windowHeight() {
      return this.$store.state.windowHeight
    },
  },
  mounted() {
    this.updateTableData()
  },
  methods: {
    updateTableData(currentPage = 1, pageSize = this.tablePage.pageSize) {
      const params = { page: currentPage, page_size: pageSize }
      if (this.ci_id) {
        getCiTriggersByCiId(this.ci_id, params).then((res) => {
          this.tableData = res.items.map((item) => {
            return {
              ...item,
              trigger: res.id2trigger[item.trigger_id],
            }
          })
        })
      } else {
        getCiTriggers(params).then((res) => {
          this.tableData = res?.result || []
          this.tablePage = {
            ...this.tablePage,
            currentPage: res.page,
            pageSize: res.page_size,
            totalResult: res.numfound,
          }
        })
      }
    },
    pageOrSizeChange(currentPage, pageSize) {
      this.updateTableData(currentPage, pageSize)
    },
  },
}
</script>

<style></style>
