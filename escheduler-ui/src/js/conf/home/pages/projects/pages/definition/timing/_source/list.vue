<template>
  <div>
    <div class="conditions-box">
      <!--<m-conditions @on-conditions="_onConditions"></m-conditions>-->
    </div>
    <div class="list-model" v-if="!isLoading">
      <template v-if="list.length">
        <div class="table-box">
          <table>
            <tr>
              <th>
                <span>{{$t('编号')}}</span>
              </th>
              <th>
                <span>{{$t('流程名称')}}</span>
              </th>
              <th>
                <span>{{$t('开始时间')}}</span>
              </th>
              <th>
                <span>{{$t('结束时间')}}</span>
              </th>
              <th>
                <span>{{$t('crontab')}}</span>
              </th>
              <th>
                <span>{{$t('失败策略')}}</span>
              </th>
              <th>
                <span>{{$t('状态')}}</span>
              </th>
              <th>
                <span>{{$t('创建时间')}}</span>
              </th>
              <th>
                <span>{{$t('更新时间')}}</span>
              </th>
              <th width="80">
                <span>{{$t('操作')}}</span>
              </th>
            </tr>
            <tr v-for="(item, $index) in list" :key="item.id">
              <td>
                <span>{{parseInt(pageNo === 1 ? ($index + 1) : (($index + 1) + (pageSize * (pageNo - 1))))}}</span>
              </td>
              <td>
                <span><a href="javascript:">{{item.processDefinitionName}}</a></span>
              </td>
              <td>
                <span>{{item.startTime | formatDate}}</span>
              </td>
              <td>
                <span>{{item.endTime | formatDate}}</span>
              </td>
              <td>
                <span>{{item.crontab}}</span>
              </td>
              <td>
                <span>{{item.failureStrategy}}</span>
              </td>
              <td>
                <span>{{_rtReleaseState(item.releaseState)}}</span>
              </td>
              <td>
                <span>{{item.createTime | formatDate}}</span>
              </td>
              <td>
                <span>{{item.updateTime | formatDate}}</span>
              </td>
              <td>
                <x-button
                        type="info"
                        shape="circle"
                        size="xsmall"
                        data-toggle="tooltip"
                        :title="$t('编辑')"
                        @click="_editTiming(item)"
                        icon="iconfont icon-bianji"
                        :disabled="item.releaseState === 'ONLINE'" >
                </x-button>
                <x-button
                        type="success"
                        shape="circle"
                        size="xsmall"
                        data-toggle="tooltip"
                        :title="$t('上线')"
                        @click="_online(item)"
                        icon="iconfont icon-erji-xiaxianjilu-copy"
                        v-if="item.releaseState === 'OFFLINE'">
                </x-button>
                <x-button
                        type="warning"
                        shape="circle"
                        size="xsmall"
                        data-toggle="tooltip"
                        :title="$t('下线')"
                        icon="iconfont icon-erji-xiaxianjilu"
                        @click="_offline(item)"
                        v-if="item.releaseState === 'ONLINE'">
                </x-button>
              </td>
            </tr>
          </table>
        </div>
        <div class="page-box">
          <x-page :current="pageNo" :total="total" show-elevator @on-change="_page"></x-page>
        </div>
      </template>
      <template v-if="!list.length">
        <m-no-data></m-no-data>
      </template>
    </div>
    <m-spin :is-spin="isLoading"></m-spin>
  </div>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import '@/module/filter/formatDate'
  import mSpin from '@/module/components/spin/spin'
  import mTiming from '../../pages/list/_source/timing'
  import mNoData from '@/module/components/noData/noData'
  import { publishStatus } from '@/conf/home/pages/dag/_source/config'

  export default {
    name: 'list',
    data () {
      return {
        isLoading: false,
        total: null,
        pageNo: 1,
        pageSize: 10,
        list: []
      }
    },
    props: {
    },
    methods: {
      ...mapActions('dag', ['getScheduleList', 'scheduleOffline', 'scheduleOnline', 'getReceiver']),
      /**
       * return state
       */
      _rtReleaseState (code) {
        return _.filter(publishStatus, v => v.code === code)[0].desc
      },
      /**
       * page
       */
      _page (val) {
        this.pageNo = val
        this._getScheduleList()
      },
      /**
       * Inquire list
       */
      _getScheduleList (flag) {
        this.isLoading = !flag
        this.getScheduleList({
          processDefinitionId: this.$route.params.id,
          searchVal: '',
          pageNo: this.pageNo,
          pageSize: this.pageSize
        }).then(res => {
          this.list = res.data.totalList
          this.total = res.data.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      },
      /**
       * search
       */
      _onConditions (o) {
        this.searchVal = o.searchVal
        this.pageNo = 1
        this._getScheduleList('false')
      },
      /**
       * online
       */
      _online (item) {
        this.pageNo = 1
        this.scheduleOnline({
          id: item.id
        }).then(res => {
          this.$message.success(res.msg)
          this._getScheduleList('false')
        }).catch(e => {
          this.$message.error(e.msg || '')
        })
      },
      /**
       * offline
       */
      _offline (item) {
        this.pageNo = 1
        this.scheduleOffline({
          id: item.id
        }).then(res => {
          this.$message.success(res.msg)
          this._getScheduleList('false')
        }).catch(e => {
          this.$message.error(e.msg || '')
        })
      },
      /**
       * get email
       */
      _getReceiver (id) {
        return new Promise((resolve, reject) => {
          this.getReceiver({ processDefinitionId: id }).then(res => {
            resolve({
              receivers: res.receivers && res.receivers.split(',') || [],
              receiversCc: res.receiversCc && res.receiversCc.split(',') || []
            })
          })
        })
      },
      /**
       * timing
       */
      _editTiming (item) {
        let self = this
        this._getReceiver(item.processDefinitionId).then(res => {
          let modal = this.$modal.dialog({
            closable: false,
            showMask: true,
            escClose: true,
            className: 'v-modal-custom',
            transitionName: 'opacityp',
            render (h) {
              return h(mTiming, {
                on: {
                  onUpdate () {
                    self.pageNo = 1
                    self._getScheduleList('false')
                    modal.remove()
                  },
                  close () {
                    modal.remove()
                  }
                },
                props: {
                  item: item,
                  receiversD: res.receivers,
                  receiversCcD: res.receiversCc
                }
              })
            }
          })
        })
      }
    },
    watch: {},
    created () {
      this._getScheduleList()
    },
    mounted () {},
    components: { mSpin, mNoData }
  }
</script>

<style lang="scss" rel="stylesheet/scss">
</style>
