<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 头部警告区域 -->
      <el-alert
        title="注意：只允许为第三级分类参数设置相关参数！"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!-- 选择商品分类的区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="{
              expandTrigger: 'hover',
              value: 'cat_id',
              label: 'cat_name',
              children: 'children',
            }"
            @change="handleSelectedCateChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
            >添加参数</el-button
          >
          <el-table :data="manyTableData" stripe>
            <el-table-column type="expand">
              <template v-slot="slotProps">
                <el-tag
                  v-for="(value, index) in slotProps.row.attr_vals"
                  :key="index"
                  closable
                  @close="deletePramItemInfo(index, slotProps.row)"
                >
                  {{ value }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="slotProps.row.inputVisible"
                  v-model="slotProps.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(slotProps.row)"
                  @blur="handleInputConfirm(slotProps.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(slotProps.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"> </el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template v-slot="slotProps">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(slotProps.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteParamInfo(slotProps.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
            >添加属性</el-button
          >
          <el-table :data="onlyTableData" stripe>
            <el-table-column type="expand">
              <template v-slot="slotProps">
                <el-tag
                  v-for="(value, index) in slotProps.row.attr_vals"
                  :key="index"
                  closable
                  @close="deletePramItemInfo(index, slotProps.row)"
                >
                  {{ value }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="slotProps.row.inputVisible"
                  v-model="slotProps.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(slotProps.row)"
                  @blur="handleInputConfirm(slotProps.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(slotProps.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"> </el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template v-slot="slotProps">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(slotProps.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteParamInfo(slotProps.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      @closed="addDialogClosed"
    >
      <el-form
        :model="addPramsForm"
        :rules="addPramsFormRules"
        ref="addPramsFormRulesRef"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addPramsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addPramsFormInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!--编辑参数的对话框  -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      @closed="editDialogClosed"
    >
      <el-form
        :model="editPramsForm"
        :rules="editPramsFormRules"
        ref="editPramsFormRulesRef"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editPramsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editPramsFormInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      cateList: [],
      selectedCateKeys: [],
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      addDialogVisible: false,
      addPramsForm: {
        attr_name: ''
      },
      addPramsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editPramsForm: {},
      editPramsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      currentAttrId: ''
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) { return this.$message.error('获取分类列表失败') }
      this.cateList = res.data
    },
    handleSelectedCateChange () {
      this.getParamsData()
    },
    handleTabClick () {
      this.getParamsData()
    },
    async getParamsData () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: {
          sel: [this.activeName]
        }
      })
      if (res.meta.status !== 200) { return this.$message.error('获取参数列表失败') }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 添加el-tag控制参数
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    addDialogClosed () {
      this.$refs.addPramsFormRulesRef.resetFields()
    },
    addPramsFormInfo () {
      this.$refs.addPramsFormRulesRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addPramsForm.attr_name,
          attr_sel: [this.activeName]
        })
        if (res.meta.status !== 201) { return this.$message.error('添加失败') }
        this.$message.success('添加成功')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    async showEditDialog (id) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
        params: {
          attr_sel: [this.activeName]
        }
      })
      if (res.meta.status !== 200) { return this.$message.error('获取参数信息失败') }
      this.editPramsForm = res.data
      this.editDialogVisible = true
      this.currentAttrId = id
    },
    editPramsFormInfo () {
      this.$refs.editPramsFormRulesRef.validate(async valid => {
        if (!valid) { }
        const { data: res } = await this.$http.put(`categories/${this.cateId}`, {
          attr_name: this.editPramsForm.attr_name,
          attr_sel: [this.activeName]
        })
        if (res.meta.status !== 200) { return this.$message.error('更新失败') }
        this.$message.success('更新成功')
        this.editDialogVisible = false
        this.getParamsData()
      })
    },
    async deleteParamInfo (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
      if (res.meta.status !== 200) { this.$message.error('删除失败') }
      this.$message.success('删除成功')
      this.getParamsData()
    },
    handleInputConfirm (info) {
      if (info.inputValue.trim().length === 0) {
        info.inputValue = ''
        info.inputVisible = false
        return
      }
      info.attr_vals.push(info.inputValue.trim())
      this.saveAttrVals(info)
      info.inputValue = ''
      info.inputVisible = false
    },
    showInput (info) {
      info.inputVisible = true
      this.$nextTick(() => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    deletePramItemInfo (index, info) {
      info.attr_vals.splice(index, 1)
      this.saveAttrVals(info)
    },
    async saveAttrVals (info) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${info.attr_id}`, {
        attr_name: info.attr_name,
        attr_sel: [this.activeName],
        attr_vals: info.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        this.$message.error('更新失败')
      } else {
        this.$message.success('更新成功')
      }
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectedCateKeys.length !== 3) { return true }
      return false
    },
    titleText () {
      if (this.activeName === 'many') { return '动态参数' }
      return '静态属性'
    },
    cateId () {
      return this.selectedCateKeys[this.selectedCateKeys.length - 1]
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
