<template>
  <m-list-construction :title="$t('Tenant Manage')">
    <template slot="conditions">
      <m-conditions @on-conditions="_onConditions">
        <template slot="button-group">
          <x-button type="ghost" size="small" @click="_create('')">{{$t('Create Tenant')}}</x-button>
        </template>
      </m-conditions>
    </template>
    <template slot="content">
      <template v-if="tenementList.length">
        <m-list @on-edit="_onEdit"
                :tenement-list="tenementList"
                :page-no="searchParams.pageNo"
                :page-size="searchParams.pageSize">

        </m-list>
        <div class="page-box">
          <x-page :current="parseInt(searchParams.pageNo)" :total="total" :page-size="searchParams.pageSize" show-elevator @on-change="_page"></x-page>
        </div>
      </template>
      <template v-if="!tenementList.length">
        <m-no-data></m-no-data>
      </template>
      <m-spin :is-spin="isLoading"></m-spin>
    </template>
  </m-list-construction>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import mList from './_source/list'
  import mSpin from '@/module/components/spin/spin'
  import mCreateTenement from './_source/createTenement'
  import mNoData from '@/module/components/noData/noData'
  import listUrlParamHandle from '@/module/mixin/listUrlParamHandle'
  import mConditions from '@/module/components/conditions/conditions'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'tenement-index',
    data () {
      return {
        total: null,
        isLoading: true,
        tenementList: [],
        searchParams: {
          pageSize: 10,
          pageNo: 1,
          searchVal: ''
        }
      }
    },
    mixins: [listUrlParamHandle],
    props: {},
    methods: {
      ...mapActions('security', ['getTenantListP']),
      /**
       * Query
       */
      _onConditions (o) {
        this.searchParams = _.assign(this.searchParams, o)
        this.searchParams.pageNo = 1
      },
      _page (val) {
        this.searchParams.pageNo = val
      },
      _onEdit (item) {
        this._create(item)
      },
      _create (item) {
        let self = this
        let modal = this.$modal.dialog({
          closable: false,
          showMask: true,
          escClose: true,
          className: 'v-modal-custom',
          transitionName: 'opacityp',
          render (h) {
            return h(mCreateTenement, {
              on: {
                onUpdate () {
                  self._debounceGET('false')
                  modal.remove()
                },
                close () {
                  modal.remove()
                }
              },
              props: {
                item: item
              }
            })
          }
        })
      },
      _getList (flag) {
        this.isLoading = !flag
        this.getTenantListP(this.searchParams).then(res => {
          this.tenementList = []
          this.tenementList = res.totalList
          this.total = res.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      }
    },
    watch: {
      // router
      '$route' (a) {
        // url no params get instance list
        this.searchParams.pageNo = _.isEmpty(a.query) ? 1 : a.query.pageNo
      }
    },
    created () {
    },
    mounted () {
    },
    components: { mList, mListConstruction, mConditions, mSpin, mNoData }
  }
</script>
