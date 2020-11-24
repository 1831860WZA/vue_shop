<template>
<div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
        <!-- 添加角色按钮区域 -->
        <el-row>
            <el-col>
                <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>

        <!-- 角色列表区域 -->
        <el-table :data="rolelist" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                            <el-tag closable @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 渲染二级权限和三级权限 -->
                        <el-col :span="19">
                            <!-- 通过 for 循环 嵌套渲染二级权限 -->
                            <el-row :class="[i2 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                <el-col :span="6">
                                    <el-tag closable @close="removeRightById(scope.row, item2.id)" type="success">{{ item2.authName }}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <el-col :span="18">
                                    <el-tag closable @close="removeRightById(scope.row, item3.id)" type="warning" v-for="item3 in item2.children" :key="item3.id">{{ item3.authName }}</el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                    <!-- <pre>
              {{scope.row}}
            </pre> -->
                </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                    <!-- 修改按钮 -->
                    <el-button type="primary" size="mini" icon="el-icon-edit" @click="rolesEditDialog(scope.row.id)">编辑</el-button>
                    <!-- 删除按钮 -->
                    <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeRole(scope.row.id)">删除</el-button>
                    <!-- 分配权限按钮 -->
                    <el-button type="setting" size="mini" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>

    <!-- 编辑时出现的对话框 -->
    <el-dialog title="修改角色" :visible.sync="roleDialogVisible" width="50%">
        <el-form :model="roleForm" ref="rolelistRef" :rules="roleFormRules">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="roleForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="roleForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer">
            <el-button @click="roleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="confirmRole">确 定</el-button>
        </span>
    </el-dialog>

    <!-- 添加角色时出现的对话框 -->
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="closeAddRole">
        <el-form :model="addRoleForm" ref="addRolelistRef" :rules="addRoleFormRules">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="addRoleForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="addRoleForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer">
            <el-button @click="addRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="confirmAddRole">确 定</el-button>
        </span>
    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
        <!-- 树形控件 -->
        <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
        <span slot="footer">
            <el-button @click="setRightDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    name: 'Roles',
    data() {
        return {
            // 所有角色列表数据
            rolelist: [],
            roleDialogVisible: false,
            // 保存点击编辑按钮的表单数据
            roleForm: {},
            // 编辑按钮的表单验证
            roleFormRules: {
                roleName: [{
                        required: true,
                        message: '请输入角色名称',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 10,
                        message: '长度在2~10个字符',
                        trigger: 'blur'
                    },
                ],
                roleDesc: [{
                        required: true,
                        message: '请输入角色描述',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 10,
                        message: '长度在2~10个字符',
                        trigger: 'blur'
                    },
                ],
            },
            addRoleDialogVisible: false,
            addRoleForm: {
                roleName: '',
                roleDesc: '',
            },
            // 添加按钮的表单验证
            addRoleFormRules: {
                roleName: [{
                        required: true,
                        message: '请输入角色名称',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 10,
                        message: '长度在2~10个字符',
                        trigger: 'blur'
                    },
                ],
                roleDesc: [{
                        required: true,
                        message: '请输入角色描述',
                        trigger: 'blur'
                    },
                    {
                        min: 2,
                        max: 10,
                        message: '长度在2~10个字符',
                        trigger: 'blur'
                    },
                ],
            },
            // 控制分配权限对话框的显示与隐藏
            setRightDialogVisible: false,
            rightslist: [],
            // 树形控件的属性绑定对象
            treeProps: {
                label: 'authName',
                children: 'children',
            },
            // 默认选中的节点Id值数组
            defKeys: [],
            // 当前即将分配权限的角色id
            roleId: ''
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        // 请求角色列表的数据
        async getRolesList() {
            const {
                data: res
            } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败!')
            }
            this.rolelist = res.data
        },
        // 点击编辑按钮请求的数据
        async rolesEditDialog(id) {
            this.roleDialogVisible = true
            // console.log(id);
            const {
                data: res
            } = await this.$http.get('roles/' + id)
            // console.log(res);
            if (res.meta.status !== 200) {
                return this.$message.error('请求失败!')
            }
            this.roleForm = res.data
            this.getRolesList()
        },
        // 编辑提交角色
        confirmRole() {
            // 验证表单
            this.$refs.rolelistRef.validate(async (valid) => {
                if (!valid) return
                const {
                    data: res
                } = await this.$http.put(
                    'roles/' + this.roleForm.roleId, {
                        roleName: this.roleForm.roleName,
                        roleDesc: this.roleForm.roleDesc,
                    }
                )
                if (res.meta.status !== 200) {
                    return this.$message.error('提交失败！')
                }
                // 隐藏消息框
                this.roleDialogVisible = false
                this.$message.success('提交成功！')
                // 重新请求数据渲染页面
                this.getRolesList()
            })
        },
        // 删除角色
        async removeRole(id) {
            const confirmResult = await this.$confirm(
                '此操作将永久删除该文件, 是否继续?',
                '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                }
            ).catch((err) => err)
            // console.log(confirmResult);  'confirm'
            if (confirmResult !== 'confirm') {
                this.$message.info('您已取消删除操作')
            } else {
                const {
                    data: res
                } = await this.$http.delete('roles/' + id)
                if (res.meta.status !== 200) return
                this.$message.success('删除成功')
                this.getRolesList()
            }
        },
        // 添加角色
        confirmAddRole() {
            this.$refs.addRolelistRef.validate(async (valid) => {
                if (!valid) return
                const {
                    data: res
                } = await this.$http.post('roles', {
                    roleName: this.addRoleForm.roleName,
                    roleDesc: this.addRoleForm.roleDesc,
                })
                if (res.meta.status !== 201) {
                    return this.$message.error('创建失败!')
                }
                this.addRoleDialogVisible = false
                this.$message.success('创建成功!')
                this.getRolesList()
            })
        },
        // 关闭添加页面时重置表单
        closeAddRole() {
            this.$refs.addRolelistRef.resetFields()
        },
        // 根据Id删除对应的权限
        async removeRightById(role, rightId) {
            const confirmResult = await this.$confirm(
                '此操作将永久删除该文件, 是否继续?',
                '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                }
            ).catch((err) => err)
            if (confirmResult !== 'confirm') {
                this.$message.info('取消了删除!')
            } else {
                const {
                    data: res
                } = await this.$http.delete(
                    `roles/${role.id}/rights/${rightId}`
                )
                if (res.meta.status !== 200) {
                    return this.$message.error('删除权限失败!')
                }
                role.children = res.data
                this.$message.success('删除成功!')
            }
        },
        // 展示分配权限的对话框
        async showSetRightDialog(role) {
            this.roleId = role.id
            // 获取所有数据的权限
            const {
                data: res
            } = await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.error('获取权限数据失败!')
            }
            this.rightslist = res.data
            // console.log(this.rightslist);

            this.getLeafKeys(role, this.defKeys)
            this.setRightDialogVisible = true
        },
        // 通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeys 数组中
        getLeafKeys(node, arr) {
            // 如果当前 node 节点不包含 children 属性，则是三级属性
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach((item) => this.getLeafKeys(item, arr))
        },
        setRightDialogClosed() {
            this.defKeys = []
        },
        // 点击为角色分配权限
        async allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const isStr = keys.join(',')
            const {
                data: res
            } = await this.$http.post(`roles/${this.roleId}/rights`, {
                rids: isStr
            })

            if (res.meta.status !== 200) {
                return this.$message.error('分配权限失败!')
            }

            this.$message.success('分配权限成功!')
            this.getRolesList();
            this.setRightDialogVisible = false;
        }
    },
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}

.bdtop {
    border-top: 1px solid #eee;
}

.bdbottom {
    border-bottom: 1px solid #eee;
}

.vcenter {
    display: flex;
    align-items: center;
}
</style>
