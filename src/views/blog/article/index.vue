<template>
  <div class="app-container">
    <el-container>
      <el-aside width="10%">
        <el-input v-model="filterCategoryName" placeholder="分类名" size="mini" clearable />
        <el-tree ref="categoryTree" :data="categoryList" :filter-node-method="filterCategoryNode" empty-text="None" default-expand-all highlight-current @node-click="getNowCategoryArticles" />
      </el-aside>
      <el-container>
        <el-header height="30">
          <el-button type="default" size="mini" @click="refreshList">刷新</el-button>
          <el-button type="danger" size="mini" @click="batchDelete">批量删除</el-button>
          <router-link :to="{path: '/blog/article/create'}"><el-button type="primary" size="mini">创建</el-button></router-link>
        </el-header>
        <el-main>
          <el-table v-loading="loadingIcon" :data="articleList" :element-loading-text="loadingText" tooltip-effect="dark" element-loading-spinner="el-icon-loading" border style="width: 100%" size="small" @selection-change="handleSelectionChange" @sort-change="sortByCustom">
            <el-table-column type="selection" width="50" />
            <el-table-column prop="id" label="ID" width="50" sortable />
            <el-table-column prop="category_name" label="Category" width="80" show-overflow-tooltip />
            <el-table-column prop="user_name" label="User" width="80" show-overflow-tooltip />
            <el-table-column prop="title" label="Title" width="200" show-overflow-tooltip>
              <template slot-scope="scope">
                <a :href="'https://www.lnmpa.top/' + scope.row.slug" target="_blank">{{ scope.row.title }}</a>
              </template>
            </el-table-column>
            <el-table-column label="Description" show-overflow-tooltip>
              <template slot-scope="scope">
                <p>{{ scope.row.description }}</p>
              </template>
            </el-table-column>
            <el-table-column label="草稿" width="51">
              <template slot-scope="scope">
                <el-button v-if="scope.row.is_draft == true" type="danger" size="mini" @click="draftArticle(scope.row, 0)">Y</el-button>
                <el-button v-if="scope.row.is_draft == false" type="success" size="mini" @click="draftArticle(scope.row, 1)">N</el-button>
              </template>
            </el-table-column>
            <el-table-column prop="updated_at" label="Update At" width="142" sortable>
              <template slot-scope="scope">
                <small><i class="el-icon-time" />{{ scope.row.updated_at }}</small>
                <small><i class="el-icon-timer" />{{ scope.row.created_at }}</small>
              </template>
            </el-table-column>
            <el-table-column fixed="right" label="操作" width="186">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" @click="showArticleContent(scope.row)">预览</el-button>
                <el-button type="warning" size="mini">
                  <router-link :to="{path: '/blog/article/edit/'+scope.row.id}">编辑</router-link>
                </el-button>
                <el-button type="danger" size="mini" @click="deleteArticle(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-main>
        <el-footer height="0">
          <el-pagination :total="listTotal" :current-page.sync="listCurrent" :page-sizes="listPageSelect" :page-size.sync="listPerPage" layout="total, sizes, prev, pager, next, jumper" background prev-text="上一页" next-text="下一页" @size-change="clickChangePerPage" @current-change="clickChangeCurrentPage" />
        </el-footer>
      </el-container>
    </el-container>
    <!--预览 content-->
    <el-dialog title="预览" :visible.sync="articleContentVisible">
      <div v-html="articleContentValue" />
      <span slot="footer" class="dialog-footer"><el-button type="primary" size="mini" @click="articleContentVisible = false">关 闭</el-button></span>
    </el-dialog>
  </div>
</template>

<script>
import { articleList, articleDelete, batchDeleteArticle, articleDetail, draftArticle, categoryList } from '@/api'
export default {
  name: 'ArticleIndex',
  data() {
    return {
      loadingIcon: true,
      loadingText: '数据加载中...',
      articleList: [],
      multipleSelected: [],
      listTotal: 0,
      listCurrent: 1,
      listPageSelect: [10, 20, 50, 100, 200],
      listPerPage: 10,
      articleContentVisible: false,
      articleContentValue: '',
      sortWay: 'DESC',
      sortField: 'id',
      categoryList: [{ id: 0, label: '所有分类', children: [] }],
      filterCategoryName: '',
      // 默认显示所有分类下的文章
      nowCategoryId: 0
    }
  },
  watch: {
    filterCategoryName(val) {
      this.$refs.categoryTree.filter(val)
    }
  },
  created() {
    this.loadingIcon = true
    this.getCategoryList()
    this.articlePageList()
  },
  methods: {
    refreshList() {
      this.loadingIcon = true
      this.articlePageList()
    },
    sortByCustom(column) {
      // 后端排序
      this.sortField = column.prop
      if (column.order === 'ascending') {
        this.sortWay = 'ASC'
      } else if (column.order === 'descending') {
        this.sortWay = 'DESC'
      } else {
        return false
      }
      // TODO 待完成
      const sortCOndition = {
        sortField: this.sortField,
        sortWay: this.sortWay
      }
      this.articlePageList(sortCOndition)
    },
    getCategoryList() {
      categoryList().then(response => {
        const categories = []
        for (const i in response.data) {
          categories.push({ label: response.data[i].name, id: response.data[i].id })
        }
        this.categoryList[0].children = categories
      }).catch(error => {
        console.error(error)
      })
    },
    filterCategoryNode(value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    },
    getNowCategoryArticles(nowCate, node, self) {
      this.nowCategoryId = parseInt(nowCate.id)
      this.articlePageList()
    },
    articlePageList(extras = {}) {
      const params = { per_page: this.listPerPage, page: this.listCurrent, category_id: null }
      if (extras.sortField && extras.sortWay) {
        params.sort_field = extras.sortField
        params.sort_way = extras.sortWay
      }
      if (this.nowCategoryId !== 0) {
        params.category_id = this.nowCategoryId
      }

      articleList(params).then(response => {
        if (response.status === true) {
          this.articleList = response.data
          this.listTotal = response.meta.pagination.total
          this.listCurrent = response.meta.pagination.current_page
          this.listPerPage = response.meta.pagination.per_page
        }
        this.loadingIcon = false
      }).catch(error => {
        console.error(error)
      })
    },
    handleSelectionChange(val) {
      this.multipleSelected = []
      for (var i in val) {
        this.multipleSelected.push(val[i].id)
      }
    },
    batchDelete() {
      if (this.multipleSelected.length === 0) {
        this.$message.warning('Please select at least one row. ')
        return false
      }
      this.$confirm('确定要删除 ' + this.multipleSelected.length + ' 篇文章吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        batchDeleteArticle({ ids: this.multipleSelected }).then(response => {
          if (response.status === true) {
            this.$message.success('Delete Successful')
            this.refreshList()
          }
        }).catch(() => {
          return true
        })
      }).catch(() => {
        this.$message.info('已取消')
      })
    },
    deleteArticle(row) {
      this.$confirm('Confirm Delete', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        articleDelete(row.id).then((response) => {
          if (response.status === true) {
            const index = this.articleList.indexOf(row)
            this.articleList.splice(index, 1)
            this.$message.success('Delete Successful')
          }
        })
      }).catch(() => {
        return true
      })
    },
    clickChangePerPage(val) {
      this.listPerPage = val
      this.refreshList()
    },
    clickChangeCurrentPage(val) {
      this.listCurrent = val
      this.refreshList()
    },
    showArticleContent(row) {
      articleDetail(row.id).then(response => {
        this.articleContentVisible = true
        this.articleContentValue = response.data.content
      })
    },
    draftArticle(row, draft) {
      draftArticle({ id: row.id, draft: draft }).then(response => {
        if (response.status === true) {
          this.$message.success('successful')
          this.refreshList()
        }
      })
    }
  }
}
</script>
