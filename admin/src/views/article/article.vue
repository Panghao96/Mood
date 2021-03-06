<template>
    <div class="article">
        <h2 class="tit">文章列表 ({{total}})</h2>
        
        <el-table :data="data">
            <el-table-column label="Title">
                <template slot-scope="scope">
                    <p>{{scope.row.title}}</p>
                </template>
            </el-table-column>
            <el-table-column label="Date" width=140 class="hidden">
                <template slot-scope="scope">
                    <span>{{scope.row.time.time}} {{scope.row.time.month.en}} {{scope.row.time.day.on}}</span>
                </template>
            </el-table-column>

            <el-table-column label="options" width=100>
                <template slot-scope="scope">
                    <el-tooltip class="item" effect="dark" content="View Article" placement="top">
                        <i class="el-icon-view" @click="view(scope.row.id)"></i>
                    </el-tooltip>
                    <el-tooltip effect="dark" content="Edit Article" placement="top">
                        <i class="el-icon-edit" @click="edit(scope.row.id)"></i>
                    </el-tooltip>
                    <el-tooltip effect="dark" content="Delete" placement="top">
                        <i class="el-icon-delete" @click="remove(scope.row)"></i>
                    </el-tooltip>
                </template>
            </el-table-column>

        </el-table>

        <el-pagination
            background
            :page-size="count"
            :pager-count="11"
            :total="total"
            :current-page="page"
            @current-change="load"
            layout="prev, pager, next"
        >
        </el-pagination>
    </div>
</template>

<script>
import { mapState } from 'vuex'
export default {
    data() {
        return {
            data: [],
            count: 10,
            total: 0,
            page: 1,
            loading: ''
        }
    },
    created(){
        this.total = this.$data.articleQty
        this.load();
    },
    mounted(){
        document.querySelector('.content').style.overflow = 'hidden'
    },
    destroyed(){
        document.querySelector('.content').style.overflow = 'auto'
    },
    computed: {
        ...mapState(['$data'])
    },
    methods: {
        load(page){
            /**
             * vuex 存在当前页数据
             */
            const article = this.$store.state.article;
            if(article[page]){
                this.data = article[page];
                return
            }

            this.loading = this.$loading({target: '.container'})

            this.$http.get('/article', {
                params: {page}
            }).then(res => {
                setTimeout(() => {
                    const data = res.data.body;
                    const item = ['data', 'total', 'page']

                    item.map(i => this[i] = data[i])
                    /**
                     * 添加数据到vuex，请求优化
                     */
                    this.$store.commit('setCache', {
                        type: 'article',
                        page: page || 1,
                        data: this.data,
                        total: this.total
                    })
                    this.loading.close()
                }, 500)
            })
        },
        edit(id){
            this.$router.push({
                name: 'info',
                query: { id: id }
            })
        },
        // 新窗口打开文章
        view(id){
            window.open(`${window.location.origin}/${id}`)
        },
        remove(item){
            this.$confirm('删除该文章, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http.delete(`article/${item._id}`).then(res => {
                    setTimeout(() => {
                        this.$store.commit('resetCache', 'article')
                        this.load()
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        })
                        this.$infoUpdate() // 刷新状态
                    }, 1000)
                })
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });
            });
        }
    }
}
</script>

<style lang="scss">
.article{
    height: 100%;
    display: flex;
    flex-direction: column;
    .header{
        h1{
            border-left: 2px solid #0084ff;
            padding-left: 16px;
            font-size: 18px;
            font-weight: 400;
            margin: 20px 0 30px;
            color: #0084ff;
        }
        .info{
            display: flex;
            margin: 15px 0;
            div{
                margin-right: 10px;
            }
        }
    }
    .article-list{
        width: 100%;
        thead tr{
            height: 40px;
        }
        tbody{
            tr{
                height: 50px;
            }
        }
    }
    i[class*=el-icon-]{
        cursor: pointer;
        margin: 2px;
        padding: 2px;
        font-size: 14px;
        color: #cfcfcf;
        transition: all .2s;
        &:hover{
            color: #0e8bff;
        }
        &.el-icon-delete:hover{
            color: red;
        }
    }
    /deep/ 
        .el-table__body-wrapper{
            height: calc(100% - 80px);
            overflow: auto;
        &::-webkit-scrollbar-track {
            background: #fff;
        }
        &::-webkit-scrollbar-thumb {
            background: #eef7ff;
        }
        &::-webkit-scrollbar {
            width: 6px;
        }
    }
}

@media screen and (max-width: 600px) {
    .article{
        padding: 0;
        box-shadow: none;
        .header h1{
            font-size: 16px;
            margin: 10px 0 20px;
        }
    }
    /deep/ .el-table__body-wrapper{
        height: calc(100vh - 212px) !important;
    }
    .el-table__header, .el-table__body{
        width: 100% !important;
        display: block;
        tbody{
            display: block;
        }
        thead{
            display: block;
            tr{
                display: flex;
                th{
                    padding: 6px 0;
                    &:nth-of-type(2){
                        display: none;
                    }
                    &:nth-of-type(1){
                        flex: 1;
                    }
                    &:nth-of-type(3){
                        width: 68px;
                        text-align: right;
                    }
                    .cell{
                        padding: 0;
                    }
                } 
            }
        }
        .el-table__row{
            display: flex;
            width: 100%;
            justify-content: space-between;
            td{
                &:nth-of-type(2){
                    display: none;
                }
                &:nth-of-type(1){
                    flex: 1;
                }
                &:nth-of-type(3){
                    width: 68px;
                    text-align: right;
                }
                div{
                    padding: 0;
                }
            }
        } 
    }
    
}
</style>