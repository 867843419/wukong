<template>
  <div v-loading="loading">
    <div v-empty="list.length === 0">
      <div class="log-items">
        <follow-record-cell v-for="(item, index) in list"
                            :item="item"
                            :crmType="crmType"
                            :index="index"
                            :key="index"
                            @on-handle="cellHandle"></follow-record-cell>
        <div class="load">
          <el-button type="text"
                     :loading="loadMoreLoading">{{loadMoreLoading ? '加载更多' : '没有更多了'}}</el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import FollowRecordCell from './components/FollowRecordCell' // 跟进记录
import { crmRecordIndex } from '@/api/customermanagement/common'
import { formatTimeToTimestamp } from '@/utils'

export default {
  /** 线索管理 的 线索详情 的 跟进记录*/
  name: 'record-log',
  components: {
    FollowRecordCell
  },
  props: {
    /** 模块ID */
    id: [String, Number],
    /** 没有值就是全部类型 有值就是当个类型 */
    crmType: {
      type: String,
      default: ''
    }
  },
  watch: {
    id: function(val) {
      this.refreshList()
    }
  },
  data() {
    return {
      loading: false,
      loadMoreLoading: true,
      isPost: false,
      page: 1,
      list: [] // 跟进记录列表
    }
  },
  computed: {},
  mounted() {
    this.$bus.on('follow-log-refresh', data => {
      if (data.type == 'record-log') {
        this.refreshList()
      }
    })

    // 分批次加载
    let dom = document.getElementById('follow-log-content')
    dom.onscroll = () => {
      let scrollOff = dom.scrollTop + dom.clientHeight - dom.scrollHeight
      //滚动条到底部的条件
      if (Math.abs(scrollOff) < 10 && this.loadMoreLoading == true) {
        if (!this.isPost) {
          this.isPost = true
          this.page++
          this.getList()
        } else {
          this.loadMoreLoading = false
        }
      }
    }

    this.getList()
  },
  activated: function() {},
  deactivated: function() {},
  methods: {
    getList() {
      this.loading = true
      crmRecordIndex({
        page: this.page,
        limit: 10,
        types: 'crm_' + this.crmType,
        types_id: this.id,
        by: 'record' // 类型（record 跟进记录，log 日志、examine审批、task 任务、event日程、默认是全部）
      })
        .then(res => {
          this.list = this.list.concat(res.data.list)
          if (res.data.list.length < 10) {
            this.loadMoreLoading = false
          } else {
            this.loadMoreLoading = true
          }
          this.loading = false
          this.isPost = false
        })
        .catch(() => {
          this.isPost = false
          this.loading = false
        })
    },
    refreshList() {
      this.page = 1
      this.list = []
      this.getList()
    },
    /**
     * 行布局删除
     */
    cellHandle(data) {
      if (data.type == 'delete') {
        this.list.splice(data.data.index, 1)
      }
    }
  },

  beforeDestroy() {
    this.$bus.off('follow-log-refresh')
  }
}
</script>

<style lang="scss" scoped>
.log-items {
  min-height: 400px;
  position: relative;
}

.load {
  color: #999;
  font-size: 13px;
  margin: 0 auto 15px;
  text-align: center;
  .el-button,
  .el-button:focus {
    color: #ccc;
    cursor: auto;
  }
}
</style>
