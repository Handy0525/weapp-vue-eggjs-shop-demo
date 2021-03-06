<template>
  <div>
    <el-table :data="mx_defaultTableData" size="mini">
      <el-table-column prop="name" label="名称" min-width="300">
        <div class="flex-y-center" slot-scope="scope">
          <safe-img :src="scope.row.thumbnail" width="30px" height="30px"></safe-img>
          <router-link class="ml-15 text-bold" :to="{name: 'goodsEdit', params: { goodsUuid: scope.row.uuid }}">
            {{scope.row.name}}
          </router-link>
        </div>
      </el-table-column>
      <el-table-column prop="salePrice" label="售价">
        <template slot-scope="scope">
          <span class="text-red text-bold">{{scope.row.salePrice}} 元</span>
        </template>
      </el-table-column>
      <el-table-column prop="categoryName" label="类别"></el-table-column>
      <el-table-column prop="unitName" label="单位"></el-table-column>
      <el-table-column prop="spec" label="规格"></el-table-column>
      <el-table-column prop="status" label="状态">
        <template slot-scope="scope">
          <span :class="$Constants.GOODS_STATUS_CLASS[scope.row.status]">
            {{$Constants.GOODS_STATUS[scope.row.status]}}
          </span>
        </template>
      </el-table-column>
      <el-table-column label="操作" fixed="right" width="100" align="center">
        <template slot-scope="scope">
          <el-button
            type="text"
            size="mini"
            @click="$router.push({ name: 'goodsEdit', params: { goodsUuid: scope.row.uuid } })"
          >
            编辑
          </el-button>
          <el-button v-if="scope.row.status === 'down'" type="text" size="mini" @click="upGoods(scope.row)">
            上架
          </el-button>
          <el-button v-if="scope.row.status === 'up'" class="danger-text-btn"
                     type="text" size="mini" @click="downGoods(scope.row)">下架
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      :totalCount="mx_defaultPagination.count"
      :currentPage.sync="mx_defaultPagination.page"
      :pageSize.sync="mx_defaultPagination.pageSize"
      :pageSizes="mx_defaultPagination.pageSizes"
      @handlePageChange="query">
    </pagination>
  </div>
</template>

<script>
  import { pageMixin, tableMixin } from '../../../utils/mixins';

  export default {
    name: 'goodsList',
    mixins: [pageMixin, tableMixin],
    components: {},
    data() {
      this[this.$Constants.REFRESH_DATA_CALLBACK_MAP] = {
        [this.$Constants.GOODS]: this.query,
        [this.$Constants.GOODS_CATEGORY]: this.query,
      };
      this[this.$Constants.APP_PAGE_TOOLS] = [
        {
          icon: 'el-icon-plus',
          content: '新增',
          callback: () => this.$router.push({ name: 'goodsAdd' }),
          type: 'primary',
        },
      ];
      return {};
    },
    methods: {
      refreshPage() {
        this.query();
      },
      query() {
        const params = this.mx_getTableParams();

        this.$api.goods.query(params).then((res) => {
          this.mx_setTableData(res);
        }).catch(() => {
        });
      },
      upGoods(goods = {}) {
        const { uuid, version } = goods;
        this.$api.goods.up({ uuid, version }).then(() => {
          this.$message({ message: '商品上架成功', type: 'success' });
          this.query();
        }).catch(() => {
        });
      },
      downGoods(goods = {}) {
        const { uuid, version } = goods;
        this.$api.goods.down({ uuid, version }).then(() => {
          this.$message({ message: '商品下架成功', type: 'success' });
          this.query();
        }).catch(() => {
        });
      },
    },
  };
</script>
