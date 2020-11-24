<template>
<div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

     <!-- 卡片视图 -->
     <el-card>
         <!-- 添加角色按钮区域 -->
         <el-row>
             <el-col>
                 <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
             </el-col>
         </el-row>

         <!-- 表格 -->
         <tree-table
            class="treeTable"
            :data="catelist" 
            :columns="columns" 
            :selection-type="false" 
            :expand-type="false" 
            show-index 
            index-text="#" 
            :show-row-hover="false">
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen"></i>
                <i class="el-icon-error" v-else style="color: red"></i>
            </template>
            <template slot="order" slot-scope="scope">
                <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
                <el-tag v-else-if="scope.row.cat_level === 1" type="success" size="mini">二级</el-tag>
                <el-tag v-else type="warning" size='mini'>三级</el-tag>
            </template>
            <template slot="opt" slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            </template>
         </tree-table>

         <!-- 分页区域 -->
         <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryinfo.pagenum"
            :page-sizes="[3, 5, 10, 15]"
            :page-size="queryinfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
         </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%"
        @close="addCateDialogClosed">
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
            <el-form-item label="分类名称" prop="cat_name">
                <el-input v-model="addCateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类">
                <el-cascader
                    expand-trigger='hover'
                    v-model="selectedKeys"
                    :options="parentCateList"
                    :props="cascaderProps"
                    @change="parentCateChanged"
                    clearable
                    change-on-select>
                </el-cascader>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    name: 'Cate',
    data() {
        return {
            // 商品分类的数据列表，默认为空
            catelist: [],
            // 查询条件
            queryinfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            // 总数据条数
            total: 0,
            // 为table指定列的定义
            columns: [
                {
                    label: '分类名称',
                    prop: 'cat_name'
                },
                {
                    label: '是否有效',
                    // 表示当前定义为模板列
                    type: 'template',
                    // 表示当前这一列使用模板名称
                    template: 'isok'
                },
                {
                    label: '排序',
                    type: 'template',
                    template: 'order'
                },
                {
                    label: '操作',
                    type: 'template',
                    template: 'opt'
                }
            ],
            // 控制添加分类对话框的显示与隐藏
            addCateDialogVisible: false,
            // 添加分类的表单数据对象
            addCateForm: {
                // 将要添加的分类的名称
                cat_name: '',
                // 父级分类的Id
                cat_pid: 0,
                // 分类的等级，默认要添加的是1级分类
                cat_level: 0
            },
            // 添加分类表单的验证规则对象
            addCateFormRules: {
                cat_name: [
                    { required: true, message: '请输入分类名称', trigger: 'blur' }
                ]
            },
            // 父级分类的列表
            parentCateList: [],
            // 选中的父级分类的Id数组
            selectedKeys: [],
            cascaderProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            }
        }
    },
    created() {
        this.getCateList();
    },
    methods: {
        // 获取数据
        async getCateList() {
            const { data: res } = await this.$http.get('categories', {
                params: this.queryinfo
            })

            if (res.meta.status !== 200) {
                return this.$messgae.error('获取商品分类失败!')
            }

            console.log(res.data);
            // 把数据列表，赋值给 catelist
            this.catelist = res.data.result;
            // 为总数据条数赋值
            this.total = res.data.total;
        },
        // 监听pagesize改变  每页显示的条数
        handleSizeChange(newSize) {
            this.queryinfo.pagesize = newSize;
            this.getCateList();
        },
        // 监听pagenum改变  当前页
        handleCurrentChange(newPage) {
            this.queryinfo.pagenum = newPage;
            this.getCateList();
        },
        // 点击按钮，展示添加分类的对话框
        showAddCateDialog() {
            this.getParentCateList();
            this.addCateDialogVisible = true
        },
        // 获取父级分类的数据列表
        async getParentCateList() {
            const { data: res } = await this.$http.get('categories', {
                params: { type: 2 }
            })
            console.log(res.data);
            this.parentCateList = res.data;
        },
        // 选择项发生变化触发这个函数
        parentCateChanged() {
            console.log(this.selectedKeys);
            if (this.selectedKeys > 0) {
                // 父级分类的Id
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1];
                // 为当前分类的等级赋值
                this.addCateForm.cat_level = this.selectedKeys.length;
            } else {
                this.addCateForm.cat_pid = 0;
                this.addCateForm.cat_level = 0;
            }
        },
        // 监听对话框的关闭事件，重置表单数据
        addCateDialogClosed() {
            this.$refs.addCateFormRef.resetFields();
            this.selectedKeys = [];
            this.addCateForm.cat_pid = 0;
            this.addCateForm.cat_level = 0;
        },
        // 点击按钮，添加新的分类
        addCate() {
            this.$refs.addCateFormRef.validate(async valid => {
                // console.log(valid);
                if (!valid) return;
                const { data: res } = await this.$http.post('categories', this.addCateForm)

                if (res.meta.status !== 201) {
                    return this.$messgae.error('添加分类失败!')
                }

                this.$message.success('添加分类成功!');
                this.getCateList();
                this.addCateDialogVisible = false;
            })
        }
    }
}
</script>

<style lang="less">
.treeTable {
    margin-top: 15px;
}

.el-cascader-panel {
    height: 300px;
}
</style>
