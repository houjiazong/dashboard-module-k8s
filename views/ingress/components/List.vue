
<template>
  <page-list
    :list="list"
    :columns="columns"
    :group-actions="groupActions"
    :single-actions="singleActions">
    <template v-slot:group-actions-append>
      <cluster-namespace :getParams.sync="list.getParams" @refresh="fetchData" class="ml-3" />
    </template>
  </page-list>
</template>

<script>
import * as R from 'ramda'
import ClusterNamespace from '@K8S/sections/ClusterNamespace'
import ColumnsMixin from '../mixins/columns'
import SingleActionsMixin from '../mixins/singleActions'
import WindowsMixin from '@/mixins/windows'
import ListMixin from '@/mixins/list'
import { getNameFilter } from '@/utils/common/tableFilter'

export default {
  name: 'K8SIngressList',
  components: {
    ClusterNamespace,
  },
  mixins: [WindowsMixin, ListMixin, ColumnsMixin, SingleActionsMixin],
  props: {
    id: String,
    getParams: {
      type: Object,
      default: () => ({}),
    },
  },
  data () {
    return {
      list: this.$list.createList(this, {
        id: this.id,
        resource: 'ingresses',
        apiVersion: 'v1',
        getParams: this.getParams,
        idKey: 'name',
        filterOptions: {
          name: getNameFilter(),
        },
      }),
      groupActions: [
        {
          label: '新建',
          permission: 'k8s_ingresses_create',
          action: () => {
            this.$router.push({ path: '/k8s-ingress/create' })
          },
          meta: () => ({
            buttonType: 'primary',
          }),
        },
        {
          label: '删除',
          permission: 'k8s_ingresses_delete',
          action: () => {
            const data = this.list.selectedItems
            const requestData = {
              cluster: data[0].clusterID,
            }
            const namespace = data[0].namespace
            if (namespace) requestData.namespace = namespace
            this.createDialog('DeleteResDialog', {
              vm: this,
              data,
              columns: this.columns,
              title: '删除',
              name: '定时任务',
              onManager: this.onManager,
              idKey: 'name',
              requestData,
            })
          },
          meta: () => {
            let validate = true
            let tooltip = ''
            if (this.list.selectedItems.length > 0) {
              let namespaces = this.list.selectedItems.map(v => v.namespace)
              let unique = Array.from(new Set(namespaces))
              if (unique.length > 1) {
                validate = false
                tooltip = '请选择同一个命名空间下的资源'
              }
            } else {
              validate = false
              tooltip = '请选择需要删除的资源，且为同一命名空间下的资源'
            }
            return {
              validate,
              tooltip,
            }
          },
        },
      ],
    }
  },
  created () {
    this.fetchData()
  },
  methods: {
    fetchData () {
      if (this.list.getParams.cluster) {
        if (this.list.getParams.all_namespace || this.list.getParams.namespace) {
          this.list.fetchData()
        }
      }
    },
    handleOpenSidepage (row) {
      this.sidePageTriggerHandle(this, 'K8SIngressSidePage', {
        id: row.name,
        resource: 'ingresses',
        getParams: () => {
          const params = R.clone(this.list.getParams)
          if (row.namespace) {
            params.namespace = row.namespace
            if (params.all_namespace) delete params.all_namespace
          }
          return params
        },
        idKey: 'name',
        apiVersion: 'v1',
      }, {
        list: this.list,
      })
    },
  },
}
</script>