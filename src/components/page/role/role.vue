<style>
    ::-webkit-scrollbar {
        display: none; /* Chrome Safari */
        scrollbar-width: none; /* Firefox */
        -ms-overflow-style: none; /* IE 10+ */
    }
</style>

<template xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
    <div>
        <!-- 搜索筛选 -->
        <el-form :inline="true" :model="formInline" class="user-search" style="padding-left: 30px;padding-top: 20px">
            <el-row>
                <el-col :span="6">
                    <el-form-item label="角色名称：">
                        <el-input size="small" v-model="formInline.roleName" placeholder="输入角色名称"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item>
                        <el-button size="small" type="primary" icon="el-icon-search" @click="search">搜索</el-button>
                        <el-button v-per="['sys:role:add']" size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">新增</el-button>
                    </el-form-item>
                </el-col>
            </el-row>
        </el-form>
        <!--列表-->
        <el-table
                :data="tableData"
                stripe
                :height="screenHeight"
                border
                style="width: 100%">
            <el-table-column
                    label="序号"
                    type="index"
                    width="60"
                    align="center">
                <template slot-scope="scope">
                    <span>{{(pageparm.currentPage - 1) * pageparm.pageSize + scope.$index + 1}}</span>
                </template>
            </el-table-column>
            <el-table-column
                    prop="name"
                    label="角色名称"
                    align="center"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="remark"
                    label="角色说明"
                    align="center"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="createDate"
                    align="center"
                    label="创建时间">
            </el-table-column>
            <el-table-column
                    fixed="right"
                    label="操作"
                    width="100">
                <template slot-scope="scope">
                    <el-button v-per="['sys:role:edit']" @click="handleEdit(scope.$index, scope.row)" type="text" size="small">编辑</el-button>
                    <el-button v-per="['sys:role:del']" @click="delRole(scope.row)" style="color: red" type="text" size="small">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页组件 -->
        <Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>
        <!-- 编辑界面 -->
        <el-dialog :title="title" :visible.sync="roleFormVisible" width="40%" @close='closeDialog("edit","roleForm")' :close-on-click-modal='false'>
            <el-form label-width="80px" ref="roleForm" :model="roleForm" :rules="rules" label-position="left">
                <el-form-item label="角色名称" prop="name">
                    <el-input size="small" v-model="roleForm.name" auto-complete="off" placeholder="请输入角色名称"></el-input>
                    <el-input size="small" v-model="roleForm.id" auto-complete="off" placeholder="请输入名称" v-if="false"></el-input>
                </el-form-item>
                <el-form-item label="角色说明" prop="remark">
                    <el-input size="small" v-model="roleForm.remark" placeholder="请输入角色说明"></el-input>
                </el-form-item>
                <el-form-item label="拥有权限">
                    <el-tree  style="overflow: auto;height: 200px" @change="$forceUpdate()"
                            :data="getMenuLists"
                            show-checkbox
                              ref="tree"
                            node-key="id"
                            @check-change="handleCheckChange"
                            :default-expanded-keys="checked"
                            :default-checked-keys="checked"
                            :props="defaultProps">
                    </el-tree>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button size="small" @click='closeDialog("edit","roleForm")'>取消</el-button>
                <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('roleForm')">保存</el-button>
            </div>
        </el-dialog>
    </div>


</template>

<script>
    import Pagination from '../../common/Pagination'
    export default {
        data() {
            var validateCheckRoleName = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('请输入角色名称'));
                } else if (value != '') {
                    var json = {
                        name:value,
                        id:this.roleForm.id
                    }
                    this.$request({
                        url:'/system/role/checkRoleName',
                        method:'post',
                        // async:false,
                        headers:{
                            'Content-Type': 'application/json',
                            'Authorization': 'Bearer '+localStorage.getItem('token')
                        },
                        data:JSON.stringify(json)
                    }).then(result=>{
                        if (result.code !=200) {
                            callback(new Error(result.message));
                        }else{
                            callback();
                        }
                    })
                } else {
                    callback();
                }
            };
            return {
                getMenuLists: [],
                defaultProps: {
                    children: 'children',
                    label: 'title'
                },


                tableData: null,
                tableLoading: true,
                checked:[],
                screenHeight: document.body.clientHeight-230, // 屏幕高度
                title: '新增角色',
                roleFormVisible: false, //控制编辑页面显示与隐藏
                loading: false, //是显示加载
                indexName:false,//路径的显示与隐藏
                options:'',
                // 分页参数
                pageparm: {
                    currentPage: 1,
                    pageSize: 10,
                    total: 10
                },
                // 请求数据参数
                formInline: {
                    page: 1,
                    limit: 10,
                    roleName: '',
                },

                // rules表单验证
                rules: {
                    name: [{ required: true,validator: validateCheckRoleName, trigger: 'blur' }],
                    remark: [
                        { required: true, message: '请输入角色说明', trigger: 'blur' }
                    ],
                },
                // 编辑与添加
                roleForm: {
                    name: '',
                    remark: '',
                    treeNode: '',
                    id: '',
                },
            };
        },

        created: function() {

        },
        // 注册组件
        components: {
            Pagination
        },
        methods: {
            // 分页插件事件
            callFather(parm) {
                this.formInline.page = parm.currentPage
                this.formInline.limit = parm.pageSize
                this.Refresh()
            },

            handleCheckChange () {
                let res = this.$refs.tree.getHalfCheckedKeys()
                let check = this.$refs.tree.getCheckedKeys()
                res=res.concat(check);
                console.log(res)
                this.roleForm.treeNode=res;
            },


            delRole(row) {
                this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    // 请求方法
                    var json = {id:row.id};
                    this.$request({
                        url:'/system/role/delRole',
                        method:'post',
                        headers:{
                            'Content-Type': 'application/json',
                            'Authorization': 'Bearer '+localStorage.getItem('token')
                        },
                        data:JSON.stringify(json)
                    }).then(result=>{
                        if (result.code === 200) {
                            this.$message.success(result.message);
                            this.Refresh()
                        }else {
                            this.$message.error(result.message);
                        }
                    })
                }).catch(() => {

                });
            },

            // 关闭编辑、增加弹出框
            closeDialog(dialog,roleForm) {
                //清楚表单验证
                this.$refs[roleForm].resetFields()
                this.indexName=false;
                if (dialog == 'edit') {
                    this.roleFormVisible = false
                }
            },
            // 编辑、添加提交方法
            submitForm(editData) {
                this.$refs[editData].validate(valid => {
                    if (valid) {
                        if (this.roleForm.id==null||this.roleForm.id==''){
                            console.log(JSON.stringify(this.roleForm))
                            // 请求方法
                            this.$request({
                                url:'/system/role/saveRole',
                                method:'post',
                                headers:{
                                    'Content-Type': 'application/json',
                                    'Authorization': 'Bearer '+localStorage.getItem('token')
                                },
                                data:JSON.stringify(this.roleForm)
                            }).then(result=>{
                                if (result.code ==200) {
                                    this.roleFormVisible = false
                                    this.$message.success(result.message);
                                    this.Refresh()
                                }else {
                                    this.$message.error(result.message);
                                }
                            })
                        }else{
                            //请求方法
                            this.$request({
                                url:'/system/role/editRole',
                                method:'post',
                                headers:{
                                    'Content-Type': 'application/json',
                                    'Authorization': 'Bearer '+localStorage.getItem('token')
                                },
                                data:JSON.stringify(this.roleForm)
                            }).then(result=>{
                                debugger

                                if (result.code == 200) {
                                    this.roleFormVisible = false
                                    this.$message.success(result.message);
                                    this.Refresh()

                                }else {
                                    this.$message.error(result.message);
                                }
                            })
                        }
                        this.loading = false
                    } else {
                        return false
                    }
                })
            },


            //显示编辑界面
            handleEdit: function(index, row) {
                this.checked=[]
                this.getMenuLists=[]
                this.getMenuList()
                this.roleFormVisible = true
                if (row != undefined && row != 'undefined') {
                    this.title = '编辑角色'//
                    var json = {id:row.id};
                    this.$request({
                        url:'/system/role/getRoleInfo',
                        method:'post',
                        headers:{
                            'Authorization': 'Bearer '+localStorage.getItem('token'),
                            'Content-Type': 'application/json'
                        },
                        data:JSON.stringify(json)
                    }).then(result=>{
                        if (result.code === 200) {
                            this.roleForm = result.data
                            this.checked=result.data.perIds

                        }else {
                            this.roleForm = result.data
                        }
                    })

                } else {
                    this.title = '新增角色'
                    this.roleForm.id="";

                }
            },

            Refresh(){
                var that = this;
                this.$request({
                    url: '/system/role/getRoleList',
                    method: 'post',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    },
                    data:JSON.stringify(this.formInline)
                }).then(function(result) {
                    if (result.code == 200) {
                        that.tableData = result.data.records;
                        // 分页赋值
                        that.pageparm.currentPage =result.data.current;
                        that.pageparm.pageSize = result.data.size;
                        that.pageparm.total = result.data.total;
                        that.tableLoading=false;

                    } else {
                        this.$message.error(result.message);
                        that.tableLoading=false;
                    }
                });
            },
            search() {
                this.Refresh()
            },
            getMenuList(){
                var that = this;
                var temp= {
                    type:'',
                    menuName:''
                }
                this.$request({
                    url: '/system/menu/getMenu',
                    method: 'post',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    },
                    data:JSON.stringify(temp)
                }).then(function(result) {
                    if (result.code == 200) {
                        that.getMenuLists = result.data;
                    } else {
                        this.$message.error(result.message);
                    }
                });
            },


        },
        mounted() {
            var that = this;
            this.$request({
                url: '/system/role/getRoleList',
                method: 'post',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                },
                data:JSON.stringify(this.formInline)
            }).then(function(result) {
                if (result.code == 200) {
                    that.tableData = result.data.records;
                    // 分页赋值
                    that.pageparm.currentPage =result.data.current;
                    that.pageparm.pageSize = result.data.size;
                    that.pageparm.total = result.data.total;
                    that.tableLoading=false;

                } else {
                    this.$message.error(result.message);
                    that.tableLoading=false;
                }
            });
        },

    };
</script>

<style>

</style>