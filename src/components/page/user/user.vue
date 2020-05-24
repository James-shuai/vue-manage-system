<style>
    ::-webkit-scrollbar {
        display: none; /* Chrome Safari */
        scrollbar-width: none; /* Firefox */
        -ms-overflow-style: none; /* IE 10+ */
    }
</style>

<template>
    <div>
        <!-- 搜索筛选 -->
        <el-form :inline="true" :model="formInline" class="user-search" style="padding-left: 30px;padding-top: 20px">
            <el-row>
                <el-col :span="6">
                    <el-form-item label="用户名：">
                        <el-input size="small" v-model="formInline.username" placeholder="输入用户名"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item label="真实姓名：">
                        <el-input size="small" v-model="formInline.nickName" placeholder="输入真实姓名"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item label="手机号：">
                        <el-input size="small" v-model="formInline.mobile" placeholder="输入手机号"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item>
                        <el-button size="small" type="primary" icon="el-icon-search" @click="search">搜索</el-button>
                        <el-button v-per="['sys:user:add']" size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">新增</el-button>
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
                    prop="username"
                    label="用户名"
                    align="center"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="nickName"
                    label="真实姓名"
                    align="center"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="mobile"
                    label="手机号"
                    align="center"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="email"
                    label="邮箱"
                    align="center"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="accountNonLocked"
                    label="锁定账户"
                    align="center"
                    width="180">
                <template slot-scope="scope">
                    <el-switch v-per="['sys:user:switch']"
                            v-model="scope.row.accountNonLocked"
                            :active-value="false"
                            :inactive-value="true"
                            active-color="#ff4949"
                            inactive-color="#13ce66"
                            @change="changeSwitch(scope.row)"/>
                </template>
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
                    <el-button v-per="['sys:user:edit']" @click="handleEdit(scope.$index, scope.row)" type="text" size="small">编辑</el-button>
                    <el-button v-per="['sys:user:delete']" @click="delRole(scope.row)" style="color: red" type="text" size="small">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页组件 -->
        <Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>
        <!-- 编辑界面 -->
        <el-dialog :title="title" :visible.sync="userFormVisible" width="40%" @close='closeDialog("edit","userForm")' :close-on-click-modal='false'>
            <el-form label-width="80px" ref="userForm" :model="userForm" :rules="rules" label-position="left">
                <el-form-item label="用户名" prop="username">
                    <el-input size="small" v-model="userForm.username" auto-complete="off" placeholder="请输入用户名"></el-input>
                    <el-input size="small" v-model="userForm.id" auto-complete="off" v-if="false"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password" v-if="passwordVisible">
                    <el-input size="small" v-model="userForm.password" placeholder="请输入密码" show-password></el-input>
                </el-form-item>
                <el-form-item label="确认密码" prop="confirmPassword" v-if="passwordVisible">
                    <el-input size="small" v-model="userForm.confirmPassword" placeholder="请再次输入密码" show-password></el-input>
                </el-form-item>
                <el-form-item label="真实姓名" prop="nickName">
                    <el-input size="small" v-model="userForm.nickName" placeholder="请输入真实姓名"></el-input>
                </el-form-item>
                <el-form-item label="手机号" prop="mobile">
                    <el-input size="small" v-model="userForm.mobile" placeholder="请输入手机号"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input size="small" v-model="userForm.email" placeholder="请输入邮箱"></el-input>
                </el-form-item>
                <el-form-item label="锁定账户" prop="accountNonLocked" v-show="accountNonLocked">
                    <el-radio v-model="userForm.accountNonLocked" :label="false">是</el-radio>
                    <el-radio v-model="userForm.accountNonLocked" :label="true">否</el-radio>
                </el-form-item>
                <el-form-item label="选择角色" prop="roleIds">
                    <el-select v-model="userForm.roleIds" multiple filterable placeholder="请选择" style="width: 100%" @change="$forceUpdate()">
                        <el-option
                                v-for="item in options"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button size="small" @click='closeDialog("edit","userForm")'>取消</el-button>
                <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('userForm')">保存</el-button>
            </div>
        </el-dialog>
    </div>


</template>

<script>
    import Pagination from '../../common/Pagination'
    export default {
        data() {
            var validateConfirmPassword = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('请再次输入密码'));
                } else if (value !== this.userForm.password) {
                    callback(new Error('两次输入密码不一致!'));
                } else {
                    callback();
                }
            };
            var validateCheckUsername = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('请输入用户名'));
                } else if (value != '') {
                    var json = {
                        username:value,
                        id:this.userForm.id
                    }
                    this.$request({
                        url:'/system/user/checkUsername',
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
                        }else {
                            callback();
                        }
                    })
                } else {
                    callback();
                }
            };
            return {
                tableData: null,
                tableLoading: true,//列表loading
                screenHeight: document.body.clientHeight-230, // 屏幕高度
                title: '新增用户',
                userFormVisible: false, //控制编辑页面显示与隐藏
                accountNonLocked:false,//是否锁定显示与隐藏
                passwordVisible:false,//密码显示与隐藏
                loading: false, //是显示加载
                indexName:false,//路径的显示与隐藏
                options: [],
                // 分页参数
                pageparm: {
                    currentPage: 1,
                    pageSize: 10,
                    total: 10
                },
                // 列表请求数据参数
                formInline: {
                    page: 1,
                    limit: 10,
                    username: '',
                    nickName: '',
                    mobile: '',
                },

                // rules表单验证
                rules: {
                    // username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
                    username: [{ required: true,validator: validateCheckUsername, trigger: 'blur' }],
                    password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
                    confirmPassword: [{ required: true,validator: validateConfirmPassword, trigger: 'blur' }],
                    nickName: [{ required: true, message: '请输入真实姓名', trigger: 'blur' }],
                    mobile: [
                        { required: true, message: '请输入手机号', trigger: 'blur' },
                        {
                            pattern: /^1(3\d|47|5((?!4)\d)|7(0|1|[6-8])|8\d)\d{8,8}$/,
                            required: true,
                            message: '请输入正确的手机号',
                            trigger: 'blur'
                        }
                        ],
                    email: [
                        { required: true, message: '请输入邮箱', trigger: 'blur' },
                        {
                            pattern: /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/,
                            required: true,
                            message: '请输入正确的邮箱',
                            trigger: 'blur'
                        }
                    ],
                    roleIds: [{ required: true, message: '请选择角色', trigger: 'blur' }],
                },
                // 编辑与添加
                userForm: {
                    username: '',
                    password: '',
                    confirmPassword: '',
                    nickName: '',
                    mobile: '',
                    email: '',
                    accountNonLocked: true,
                    roleIds: '',
                    id: '',
                },
            };
        },

        // 注册组件
        components: {
            Pagination//分页组件
        },
        methods: {
            // 分页插件事件
            callFather(parm) {
                this.formInline.page = parm.currentPage
                this.formInline.limit = parm.pageSize
                this.Refresh()
            },
            //switch
            changeSwitch(row){
                this.$request({
                    url:'/system/user/editisAccountNonLockedUser',
                    method:'post',
                    headers:{
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer '+localStorage.getItem('token')
                    },
                    data:JSON.stringify(row)
                }).then(result=>{
                    if (result.code ==200) {
                        this.Refresh()
                        this.$message.success(result.message);
                    }else {
                        this.$message.error(result.message);
                    }
                })
            },
            getRole(){
                // 请求方法
                this.$request({
                    url:'/system/role/getRole',
                    method:'post',
                    headers:{
                        'Authorization': 'Bearer '+localStorage.getItem('token')
                    },
                }).then(result=>{
                    if (result.code ==200) {
                        this.options = result.data
                    }else {
                        this.$message.error(result.message);
                    }
                })
            },

            delRole(row) {
                this.$confirm('是否删除用户'+row.username+'?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    // 请求方法
                    var json = {id:row.id};
                    this.$request({
                        url:'/system/user/delUser',
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
            closeDialog(dialog,userForm) {
                //清楚表单验证
                this.$refs[userForm].resetFields()
                this.indexName=false;
                if (dialog == 'edit') {
                    this.userFormVisible = false
                }
            },
            // 编辑、添加提交方法
            submitForm(editData) {
                this.$refs[editData].validate(valid => {
                    if (valid) {
                        if (this.userForm.id==null||this.userForm.id==''){
                            console.log(JSON.stringify(this.userForm))
                            // 请求方法
                            this.$request({
                                url:'/system/user/saveUser',
                                method:'post',
                                headers:{
                                    'Content-Type': 'application/json',
                                    'Authorization': 'Bearer '+localStorage.getItem('token')
                                },
                                data:JSON.stringify(this.userForm)
                            }).then(result=>{
                                if (result.code ==200) {
                                    this.userFormVisible = false
                                    this.$message.success(result.message);
                                    this.Refresh()
                                }else {
                                    this.$message.error(result.message);
                                }
                            })
                        }else{
                            //请求方法
                            this.$request({
                                url:'/system/user/editUser',
                                method:'post',
                                headers:{
                                    'Content-Type': 'application/json',
                                    'Authorization': 'Bearer '+localStorage.getItem('token')
                                },
                                data:JSON.stringify(this.userForm)
                            }).then(result=>{
                                if (result.code == 200) {
                                    this.userFormVisible = false
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
                this.userFormVisible = true
                this.accountNonLocked=false;
                this.passwordVisible=false;
                this.getRole()
                if (row != undefined && row != 'undefined') {
                    this.title = '编辑用户'//
                    var json = {id:row.id};
                    this.$request({
                        url:'/system/user/getUserInfo',
                        method:'post',
                        headers:{
                            'Authorization': 'Bearer '+localStorage.getItem('token'),
                            'Content-Type': 'application/json'
                        },
                        data:JSON.stringify(json)
                    }).then(result=>{
                        if (result.code === 200) {
                            this.userForm = result.data

                        }else {
                            this.userForm = result.data
                        }
                    })

                } else {
                    this.accountNonLocked=true;
                    this.passwordVisible=true;
                    this.title = '新增用户'
                    this.userForm.id="";

                }
            },

            Refresh(){
                var that = this;
                this.$request({
                    url: '/system/user/getUserList',
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


        },
        mounted() {
            var that = this;
            this.$request({
                url: '/system/user/getUserList',
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