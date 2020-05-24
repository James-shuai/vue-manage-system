<template xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
    <div>
        <!-- 搜索筛选 -->
        <el-form :inline="true" :model="formInline" class="user-search" style="padding-left: 30px;padding-top: 20px">
            <el-row>
                <el-col :span="6">
                    <el-form-item label="菜单类型：">
                        <el-select size="small" v-model="formInline.type" placeholder="请选择">
                            <el-option label="全部" value=""></el-option>
                            <el-option label="菜单" value="1"></el-option>
                            <el-option label="按钮" value="2"></el-option>
                        </el-select>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item label="菜单名称：">
                        <el-input size="small" v-model="formInline.menuName" placeholder="输入菜单名称"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item>
                        <el-button size="small" type="primary" icon="el-icon-search" @click="search">搜索</el-button>
                        <el-button v-per="['sys:menu:add']" size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">新增</el-button>
                    </el-form-item>
                </el-col>
            </el-row>




        </el-form>
        <!--列表-->
        <el-table
                v-if="tableData!=null"
                :data="tableData"
                v-loading="tableLoading"
                style="width: 100%;margin-bottom: 20px;"
                row-key="id"
                :height="screenHeight"
                border
                default-expand-all
                :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
            <el-table-column
                    prop="title"
                    label="菜单名称"
                    align="center"
                    sortable
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="code"
                    label="权限标识符"
                    align="center"
                    sortable
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="index"
                    label="路径"
                    align="center"
                    sortable
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="type"
                    label="类型"
                    align="center"
                    sortable
                    :formatter="formatSex"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="remark"
                    label="备注"
                    align="center"
                    sortable
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
                    <el-button v-per="['sys:menu:edit']" @click="handleEdit(scope.$index, scope.row)" type="text" size="small">编辑</el-button>
                    <el-button v-per="['sys:menu:del']" @click="delMenu(scope.row)" style="color: red" type="text" size="small">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 编辑界面 -->
        <el-dialog :title="title" :visible.sync="editFormVisible" width="40%" @close='closeDialog("edit","editForm")' :close-on-click-modal='false'>
            <el-form label-width="80px" ref="editForm" :model="editForm" :rules="rules" label-position="left">
                <el-form-item label="新增类型" prop="type">
                    <el-select size="small" v-model="editForm.type" placeholder="请选择" class="userRole" @change="typeChange(),$forceUpdate()" style="width: 100%">
                        <el-option label="菜单" value="1"></el-option>
                        <el-option label="按钮" value="2"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="上级菜单" prop="parentMenu">
                    <el-select size="small" v-model="editForm.parentMenu" filterable placeholder="如果是菜单目录不用选择" class="userRole" style="width: 100%" @change="$forceUpdate()">
                        <el-option
                                v-for="item in options"
                                :key="item.id"
                                :label="item.title"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="名称" prop="menuName">
                    <el-input size="small" v-model="editForm.menuName" auto-complete="off" placeholder="请输入名称"></el-input>
                    <el-input size="small" v-model="editForm.id" auto-complete="off" placeholder="请输入名称" v-if="false"></el-input>
                </el-form-item>
                <el-form-item label="权限标识" prop="code">
                    <el-input size="small" v-model="editForm.code" placeholder="请输入权限标识符"></el-input>
                </el-form-item>
                <el-form-item v-if="indexName" label="菜单路径" prop="index">
                    <el-input size="small" v-model="editForm.index" placeholder="请输入菜单路径"></el-input>
                </el-form-item>
                <el-form-item label="排序" prop="sort">
                    <el-input size="small" v-model="editForm.sort" placeholder="请输入排序序号"></el-input>
                </el-form-item>
                <el-form-item label="备注" prop="remark">
                    <el-input size="small" v-model="editForm.remark" placeholder="请输入备注"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button size="small" @click='closeDialog("edit","editForm")'>取消</el-button>
                <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('editForm')">保存</el-button>
            </div>
        </el-dialog>
    </div>


</template>

<script>
    export default {
        data() {
            return {
                tableData: null,
                tableLoading: true,
                screenHeight: document.body.clientHeight-230, // 屏幕高度
                title: '新增菜单',
                editFormVisible: false, //控制编辑页面显示与隐藏
                loading: false, //是显示加载
                indexName:false,//路径的显示与隐藏
                options:'',
                // rules表单验证
                rules: {
                    type: [{ required: true, message: '请选择角色', trigger: 'change' }],
                    menuName: [
                        { required: true, message: '请输入菜单名称', trigger: 'blur' }
                    ],
                    // userMobile: [
                    //     { required: true, message: '请输入手机号', trigger: 'blur' },
                    //     {
                    //         pattern: /^1(3\d|47|5((?!4)\d)|7(0|1|[6-8])|8\d)\d{8,8}$/,
                    //         required: true,
                    //         message: '请输入正确的手机号',
                    //         trigger: 'blur'
                    //     }
                    // ],
                    code: [{ required: true, message: '请输入权限标识符', trigger: 'blur' }],
                    index: [{ required: true, message: '请输入菜单路径', trigger: 'blur' }],
                    // parentMenu: [{ required: true, message: '请选择上级菜单', trigger: 'change' }],
                    sort: [{ required: true, message: '请输入排序序号', trigger: 'blur' }],
                    remark: [{ required: true, message: '请输入备注', trigger: 'blur' }],
                },
                // 编辑与添加
                editForm: {
                    type: '',
                    menuName: '',
                    code: '',
                    index: '',
                    parentMenu: '',
                    sort: '',
                    remark: '',
                    id: '',
                },
                formInline: {
                    menuName: '',
                    type: '',
                },
            };
        },

        created: function() {

        },
        methods: {
            formatSex: function(row, column, cellValue) {
                if (cellValue == '1') {
                    return '菜单';
                }
                if (cellValue == '2') {
                    return '按钮';
                }
            },
            typeChange(){
                if (this.editForm.type==1){
                    this.indexName=true;
                }else if (this.editForm.type==2){
                    this.indexName=false;
                }else {
                    this.indexName=false;
                }

            },
            delMenu(row) {
                this.$confirm('此操作将永久删除该菜单或按钮, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    // 请求方法
                    var json = {id:row.id};
                    this.$request({
                        url:'/system/menu/delMenu',
                        method:'post',
                        headers:{
                            'Content-Type': 'application/json',
                            'Authorization': 'Bearer '+localStorage.getItem('token')
                        },
                        data:JSON.stringify(json)
                    }).then(result=>{
                        if (result.code === 200) {
                            this.editFormVisible = false
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
            closeDialog(dialog,editForm) {
                //清楚表单验证
                this.$refs[editForm].resetFields()
                this.indexName=false;
                if (dialog == 'edit') {
                    this.editFormVisible = false
                }
            },
            // 编辑、添加提交方法
            submitForm(editData) {
                this.$refs[editData].validate(valid => {
                    if (valid) {
                        if (this.editForm.id==null||this.editForm.id==''){
                            // 请求方法
                            this.$request({
                                url:'/system/menu/saveMenu',
                                method:'post',
                                headers:{
                                    'Content-Type': 'application/json',
                                    'Authorization': 'Bearer '+localStorage.getItem('token')
                                },
                                data:JSON.stringify(this.editForm)
                            }).then(result=>{
                                if (result.code === 200) {
                                    this.editFormVisible = false
                                    this.$message.success(result.message);
                                    this.Refresh()

                                }else {
                                    this.$message.success(result.message);
                                }
                            })
                        }else{
                            // 请求方法
                            this.$request({
                                url:'/system/menu/editMenu',
                                method:'post',
                                headers:{
                                    'Content-Type': 'application/json',
                                    'Authorization': 'Bearer '+localStorage.getItem('token')
                                },
                                data:JSON.stringify(this.editForm)
                            }).then(result=>{
                                if (result.code === 200) {
                                    this.editFormVisible = false
                                    this.$message.success(result.message);
                                    this.Refresh()

                                }else {
                                    this.$message.success(result.message);
                                }
                            })
                        }
                        this.loading = false
                    } else {
                        return false
                    }
                })
            },

            getMenu(){
                this.$request({
                    url:'/system/menu/getOptionMenu',
                    method:'post',
                    headers:{
                        'Authorization': 'Bearer '+localStorage.getItem('token')
                    },
                }).then(result=>{
                    if (result.code === 200) {
                        console.log("菜单列表"+result)
                        this.options=result.data
                    }
                })
            },

            //显示编辑界面
            handleEdit: function(index, row) {
                this.getMenu()
                this.editForm.parentMenu=""
                this.editFormVisible = true
                if (row != undefined && row != 'undefined') {
                    this.title = '编辑菜单'
                    var json = {id:row.id};
                    this.$request({
                        url:'/system/menu/getMenuInfo',
                        method:'post',
                        headers:{
                            'Authorization': 'Bearer '+localStorage.getItem('token'),
                            'Content-Type': 'application/json'
                        },
                        data:JSON.stringify(json)
                    }).then(result=>{
                        if (result.code === 200) {
                            this.editForm = result.data
                            this.editForm.menuName=result.data.title
                            this.editForm.type=String(result.data.type)
                            if (result.data.parentId==0){
                                this.editForm.parentMenu=0
                            }else {
                                this.editForm.parentMenu=result.data.parentId
                            }
                            if (result.data.type==1){
                                this.indexName=true;
                            }
                        }else {
                            this.editForm = result.data
                        }
                    })

                } else {
                    this.title = '新增菜单'
                    this.editForm.id="";

                }
            },

            Refresh(){
                var that = this;
                this.$request({
                    url: '/system/menu/getMenu',
                    method: 'post',
                    headers: {
                        'Authorization': 'Bearer ' + localStorage.getItem('token'),
                        'Content-Type': 'application/json'
                    },
                    data:JSON.stringify(this.formInline)
                }).then(function(result) {
                    if (result.code == 200) {
                        that.tableData = result.data;
                        that.tableLoading=false;
                    } else {
                        that.tableLoading=false;
                        this.$message.error(result.message);

                    }
                });
            },
            search() {
                this.Refresh()
            },


        },
        mounted() {
            // const loading = this.$loading({
            //     lock: true,
            //     text: '加载中',
            //     spinner: 'el-icon-loading',
            //     background: 'rgba(0, 0, 0, 0.7)'
            // });

            var that = this;
            this.$request({
                url: '/system/menu/getMenu',
                method: 'post',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                },
                data:JSON.stringify(this.formInline)
            }).then(function(result) {
                if (result.code == 200) {
                    that.tableData = result.data;
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