<template>
  <div class="cate">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <el-tree
        :data="list"
        node-key="cat_id"
        ref="tree"
        :props="defaultProps">
        <span class="custom-tree-node" slot-scope="{ node, data }">

          <span>{{ node.label }}</span>
          <span>
            <i class="el-icon-success" v-if="data.cat_deleted === false"></i>
            <i class="el-icon-error" v-else></i>
            <el-tag size="mini" v-if="data.cat_level === 0">一级</el-tag>
            <el-tag type="success" size="mini" v-if="data.cat_level === 1">二级</el-tag>
            <el-tag type="warning" size="mini" v-if="data.cat_level === 2">三级</el-tag>
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="() => append(data)">
              编辑
            </el-button>
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="() => remove(node, data)">
              删除
            </el-button>
           </span>


        </span>
      </el-tree>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="this.querInfo.pagenum"
        :page-sizes="[4, 6, 8, 10]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" @close="addCateDialogClosed" width="50%">

      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader expand-trigger="hover"
            :options="parentCateList"
            :props="cascaderProps"
            v-model="selectedKeys"
            @change="parentCateChanged"
            clearable>
          </el-cascader>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取消</el-button>
        <el-button @click="addCate">确定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>

export default {
    data() {
    //   const catelist = [{
    //     id: 1,
    //     label: '一级 1',
    //     children: [{
    //       id: 4,
    //       label: '二级 1-1',
    //       children: [{
    //         id: 9,
    //         label: '三级 1-1-1'
    //       }, {
    //         id: 10,
    //         label: '三级 1-1-2'
    //       }]
    //     }]
    //   }, {
    //     id: 2,
    //     label: '一级 2',
    //     children: [{
    //       id: 5,
    //       label: '二级 2-1'
    //     }, {
    //       id: 6,
    //       label: '二级 2-2'
    //     }]
    //   }, {
    //     id: 3,
    //     label: '一级 3',
    //     children: [{
    //       id: 7,
    //       label: '二级 3-1'
    //     }, {
    //       id: 8,
    //       label: '二级 3-2'
    //     }]
    //   }];
      return {
        querInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 5
        },
        total: 0,
        list: [],
        defaultProps: {
          children: 'children',
          label: 'cat_name'
        },
        addCateDialogVisible: false,
        addCateForm: {
          cat_name: '',
          cat_pid: 0,
          cat_level: 0
        },
        addCateFormRules: {
          cat_name: [
            { required: true, message: '请输入分类名称', trigger: 'blur' }
          ]
        },
        parentCateList: [],
        cascaderProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children',
          checkStrictly: true
        },
        selectedKeys: ''
    }
  },
  created() {
    this._getCateList()
  },
  methods: {
    async _getCateList() {
      const {data: res} = await this.$http.get('/categories', {
        params: this.querInfo
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      console.log(res.data.result)
      const data = res.data.result
      this.list = data
      console.log(this.list)
      this.total = res.data.total
    },
    remove(node, data) {
      const parent = node.parent;
      const children = parent.data.children || parent.data;
      const index = children.findIndex(d => d.id === data.id);
      children.splice(index, 1);
      console.log(data)
      console.log(data.$treeNodeId)
    },
    handleSizeChange (newSize) {
      this.querInfo.pagesize = newSize
      this._getCateList()
    },
    handleCurrentChange (newPage) {
      this.querInfo.pagenum = newPage
      this._getCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList () {
      const {data: res} = await this.$http.get('/categories', {
        params: { type: 2 }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败！')
      }

      console.log(res.data);
      this.parentCateList = res.data
    },
    parentCateChanged () {
      console.log(this.selectedKeys);

      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const {data: res} = await this.$http.post('/categories', 
          this.addCateForm
        )

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }

        this.$message.success('添加分类成功！')
        this._getCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = [],
      this.addCateForm.cat_level = 0,
      this.addCateForm.cat_pid = 0
    }
  }
}
</script>

<style lang="less" scoped>

</style>