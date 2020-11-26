<template>
<div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
        <el-row :gutter="20">
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="queryinfo.query">
                    <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                </el-input>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" @click="goAddpage">添加商品</el-button>
            </el-col>
        </el-row>

        <!-- table表格区域 -->
        <el-table :data="goodslist" border stripe>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="goods_name" label="商品名称"></el-table-column>
            <el-table-column prop="goods_price" label="商品价格(元)" width="95"></el-table-column>
            <el-table-column prop="goods_weight" label="商品重量" width="70"></el-table-column>
            <el-table-column label="创建时间" width="140">
                <template slot-scope="scope">
                    {{scope.row.add_time | dateFormat}}
                </template>
            </el-table-column>
            <el-table-column label="操作" width="130">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                    <el-button type="primary" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"></el-button>
                </template>
            </el-table-column>
        </el-table>

        <!-- 分页区域 -->
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryinfo.pagenum" :page-sizes="[5, 10, 15, 20]" :page-size="queryinfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total" background>
        </el-pagination>
    </el-card>
</div>
</template>

<script>
export default {
    name: 'List',
    data() {
        return {
            // 查询参数对象
            queryinfo: {
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            // 商品列表
            goodslist: [],
            // 总数据条数
            total: 0
        }
    },
    created() {
        this.getGoodsList();
    },
    methods: {
        async getGoodsList() {
            const {
                data: res
            } = await this.$http.get('goods', {
                params: this.queryinfo
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品列表失败!')
            }
            this.$message.success('获取商品列表成功!')
            console.log(res);
            this.goodslist = res.data.goods;
            this.total = res.data.total;
        },
        handleSizeChange(newSize) {
            this.queryinfo.pagesize = newSize;
            this.getGoodsList();
        },
        handleCurrentChange(newPage) {
            this.queryinfo.pagenum = newPage;
            this.getGoodsList();
        },
        async removeById(goodsId) {
            const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                this.$message.info('已取消删除!')
            }
            const { data: res } = await this.$http.delete('goods/'+ goodsId)
            if (res.meta.status !== 200) {
                this.$message.error('删除商品失败!')
            }
            this.getGoodsList();
            this.$message.success('删除商品成功!')
        },
        goAddpage() {
            this.$router.push('/goods/add')
        }
    }
}
</script>

<style lang="less" scoped>

</style>
