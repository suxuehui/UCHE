<!-- 用户管理 -->
<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input size="small" v-model="filters.name" @keyup.native.13="handleQuery" placeholder="用户名"></el-input>
                </el-form-item>
                <el-form-item>
                    <Button type="primary" @click="handleQuery" icon="ios-search">查询</Button>
                </el-form-item>
            </el-form>
        </el-col>
        <!--列表-->
        <el-table :max-height="windowOutHeight-260" border :data="listData" highlight-current-row v-loading="listLoading" element-loading-text="拼命加载中" 
        element-loading-spinner="el-icon-loading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="index" label="#" width="40" align="center">
            </el-table-column>
            <el-table-column type="selection" width="40" align="center">
            </el-table-column>
            <el-table-column prop="name" label="用户名" width="110" align="center">
            </el-table-column>
            <el-table-column prop="mobile" label="电话" width="110" align="center">
            </el-table-column>
            <el-table-column prop="password" label="密码"  align="center">
            </el-table-column>
            <el-table-column prop="usertype" label="用户类型" align="center" 
                :filters="[{ text: '员工', value: 'R' }, { text: '企业', value: 'C' }, { text: '个人', value: '' }]"
                :filter-method="filterTag"
                filter-placement="bottom-end">
                <template slot-scope="scope">
                    <el-tag
                      size="small"
                      :type="scope.row.usertype === 'R' ? 'primary' : scope.row.usertype === 'C' ? 'success' : 'warning'"
                      close-transition >{{scope.row.usertype == 'R' ? '员工' : scope.row.usertype == 'C' ? '企业' : '个人'}}</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="firstlogintime" label="首次登录" :formatter="dateFormatter" align="center">
            </el-table-column>
            <el-table-column prop="lastlogintime" label="最近登录" :formatter="dateFormatterSecond" align="center">
            </el-table-column>
            <el-table-column prop="logincount" label="登录次数"  align="center">
            </el-table-column>
            <el-table-column prop="isenable" label="是否有效" :formatter="formatIsactive" align="center">
            </el-table-column>
            <el-table-column fixed="right" label="操作" align="center" width="120">
                <template slot-scope="scope">
                    <Button type="primary" shape="circle" size="small" @click="handleEdit(scope.$index, scope.row)" title="编辑用户"><Icon type="edit"></Icon></Button>
                    <Button type="primary" shape="circle" size="small" @click="roleEdit(scope.$index, scope.row)" title="编辑角色权限"><Icon type="ios-people"></Icon></Button>
                    <Button type="error" shape="circle" size="small" @click="handleDel(scope.$index, scope.row)" title="删除"><Icon type="trash-a"></Icon></Button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                 :page-sizes="[15, 50, 80, 99]"
                :page-size="pageSize"
                 layout="total, sizes, prev, pager, next, jumper"
                :total="total" style="text-align: right;margin-top: 5px;">
        </el-pagination>

        <!--编辑界面-->
        <Modal title="编辑" :modal-append-to-body="false" v-model="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" :rules="editFormRules" ref="editForm">
                <el-form-item label="" prop="name" ref="name">
                    <el-input size="small" v-model="editForm.name" auto-complete="off" @blur="checkout('name',editForm.name,0)">
                        <template slot="prepend">用户名</template>
                    </el-input>
                </el-form-item>
                <el-form-item label="" prop="mobile" ref="mobile">
                    <el-input size="small" v-model="editForm.mobile" auto-complete="off">
                        <template slot="prepend">电话</template>
                    </el-input>
                </el-form-item>
                <el-form-item label="" prop="password">
                    <el-input size="small" v-model="editForm.password" auto-complete="off">
                        <template slot="prepend">密码</template>
                    </el-input>
                </el-form-item>
                <el-form-item label="用户类型" prop="usertype">
                    <el-radio-group size="small" v-model="editForm.usertype">
                        <el-radio-button label="R">员工</el-radio-button>
                        <el-radio-button label="C" disabled>企业</el-radio-button>
                        <el-radio-button label="">个人</el-radio-button>
                    </el-radio-group>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <Button @click.native="editFormVisible = false">取消</Button>
                <Button type="primary" @click.native="editSubmit" :loading="editLoading">提交</Button>
            </div>
        </Modal>

        <!--角色编辑界面-->
        <Modal title="编辑角色" :modal-append-to-body="false" v-model="editRoleInfoVisible" :close-on-click-modal="false" @close="roleEditClose">
            <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选</el-checkbox>
            <div style="margin: 15px 0;"></div>
            <el-checkbox-group v-model="checkedCities" class="userchedaa" @change="handleCheckedCitiesChange">
                    <el-checkbox v-for="city in cities" :label="city.id" :key="city.id">{{ city.rolename }}</el-checkbox>
            </el-checkbox-group>
            <div slot="footer" class="dialog-footer">
                <Button @click.native="editRoleInfoVisible = false">取消</Button>
                <Button type="primary" @click.native="handleNodeClick" :loading="nodeLoading">提交</Button>
            </div>
        </Modal>

    </section>
</template>
<style type="text/css">
    .disabled{color:#D7D9E2;}
    .userchedaa .el-checkbox{
        margin-right: 15px;
        margin-bottom: 15px;
        margin-left: 0;
    }
</style>
<script>
    import util from '../../../styles/js/util'
    import {getUserList,modifyUser,removeUser,getRoleList,getUserRole,editUserRole} from '../../../api/api';
    export default {
        props:['windowOutHeight'],
        data() {
            return {
                filters: {
                    domSearch:[{ select:['vin','name','model','licenseplatenum'],content:'' }],//车辆查询框
                },
                filterTextVas:'',
                activeName:'first',
                tabName:'',//存储点击的tab名
                listData: [],
                customers:[],
                total: 0,
                pageSize:15,
                curResourceId:'',
                currentPage: 1,
                listLoading: false,
                sels: [],//列表选中列
                checkoutDataT:true,//数据验证返回的布尔值true
                checkoutDataF:[],//
                treeLoading:false,
                thisInput:[],//编辑时存入row验证的值
                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editRoleInfoVisible:false,
                editFormRules: {
                },
                checkAll: false,
                nodeLoading:false,
                checkedCities: [],
                cities: [],
                curUserId:'',
                isIndeterminate: true,
                
                //编辑界面数据
                editForm: {
                    id:'',
                    name:'',
                    password:'',
                    usertype:'',
                    firstlogintime:'',
                    lastlogintime:'',
                    logincount:'',
                },
            }
        },
        watch: {
              filterTextVas(val) {
                this.$refs.treeRouseVas.filter(val);
              },
        },
        methods: {
            // 筛选过滤——组织
            filterNodeCorp(value, data) {
                    if (!value) return true;
                    return data.corpname.indexOf(value) !== -1;
            },
            // 数据重复验证
            checkout:function(p,v,index){
                if(v=="") return ;
                if (this.thisInput == v) return;//编辑时 没改输入框值
                this.checkoutDataT = true;//初始化
                      let paras={
                                para:p,
                                value:v,
                      }
                    getCheckoutOfUser(paras).then((res) => {
                    let errorInput = res.data.data.param;//保存验证失败的字段
                    if(!res.data.data.result){
                        this.$message ({
                            message : '信息输入重复！',
                            type: 'warning'
                        });
                        this.$refs[errorInput].$el.className="el-form-item is-error";//输入框标红
                        this.checkoutDataF[index]=false
                    }else{
                        this.$refs[errorInput].$el.className="el-form-item";//输入框恢复
                        this.checkoutDataF[index]=true
                    }
                });
            },


            // 编辑角色
            roleEditClose:function(){
                this.checkedCities = [];
            },
            //角色编辑显示
            roleEdit : function(index,row){
                this.editRoleInfoVisible = true;
                this.curUserId = row.id;
                //初始化角色
                getRoleList().then((res) => {
                    this.cities = res.data.data.records;
                    let para = {
                        userid:this.curUserId
                    };
                    getUserRole(para).then((res) => {
                        let carry = [];
                        res.data.data.records.forEach(function(obj){
                            carry.push(obj.roleid);
                        });
                        this.checkedCities = carry;
                    });
                });
            },
            handleCheckAllChange(val) {
                let arry = [];
                this.cities.forEach(function(obj){
                    arry.push(obj.id);
                });
                    this.checkedCities = val ? arry : [];
                    this.isIndeterminate = false;
                },
                handleCheckedCitiesChange(value) {
                    let checkedCount = value.length;
                    this.checkAll = checkedCount === this.cities.length;
                    this.isIndeterminate = checkedCount > 0 && checkedCount < this.cities.length;
                },
                // 角色提交时
                handleNodeClick:function(){
                    this.nodeLoading = true;
                        let checkedNode = this.checkedCities,
                        para = [],_this = this;
                        checkedNode.forEach(function(val){
                            var obj = {};
                            obj.userid = _this.curUserId;
                            obj.roleid = val;
                            para.push(obj);
                        });
                        editUserRole(para).then((res) => {
                        var data = res.data.data;
                        if (res.data.result.code == 0) {
                            this.$message({
                              message: '编辑成功！',
                              type: 'success'
                            });
                            this.nodeLoading = false;
                            this.editRoleInfoVisible = false;
                        }
                        });
                },

            //有效转换器
            formatIsactive: function (row, column) {
                return row.isenable == 'Y' ? '有效' : row.isenable == 'N' ? '无效' : '未知';
            },
            //时间转换1
            dateFormatter: function(row,col){
                if (row.firstlogintime == "" || row.firstlogintime == undefined)  return;
                return util.formatDate.format(new Date(row.firstlogintime), 'yyyy-MM-dd');
            },
            //时间转换2
            dateFormatterSecond: function(row,col){
                if (row.lastlogintime == "" || row.lastlogintime == undefined)  return;
                return util.formatDate.format(new Date(row.lastlogintime), 'yyyy-MM-dd');
            },
            //切换当前页
            handleCurrentChange(val) {
                this.currentPage = val;
                this.handleQuery();
            },
            //切换每页显示数量
            handleSizeChange(val) {
                this.pageSize = val;
                this.handleQuery();
            },
            //获取保单列表
            handleQuery() {
                let para = {
                    currentPage : this.currentPage,
                    showCount   : this.pageSize,
                    name:this.filters.name,
                };
                this.listLoading = true;
                getUserList(para).then((res) => {
                    this.total = res.data.data.totalResult;
                    this.listData = res.data.data.records;
                    this.listLoading = false;
                });
            },
            
            // 用户类型过滤搜索
            filterTag(value, row) {
                return row.usertype === value;
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    let para = { 
                        id: row.id
                    };

                    removeUser(para).then((res) => {
                        if (res.data.result.code == 0) {
                            this.listLoading = false;
                            this.$message ({
                                message : '删除成功',
                                type: 'success'
                            });
                            this.handleQuery();
                        }
                    });
                }).catch(() => {

                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                $(".is-error").removeClass('is-error');//清空验证时的红框
                this.editFormVisible = true;
                this.editForm = row;
                // this.editForm.usertype = (!row.usertype || row.usertype == 'R') ? 'C' : 'R';
                this.thisInput = this.editForm.name;//将当前验证的字段 已获得的值存入
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.editLoading = true;
                        let para = Object.assign({}, this.editForm);
                        modifyUser(para).then((res) => {
                            if (res.data.result.code == 0) {
                                this.editLoading = false;
                                this.$message ({
                                    message : '修改成功',
                                    type: 'success'
                                });
                                this.$refs['editForm'].resetFields();
                                this.editFormVisible = false;
                                this.handleQuery();
                            }
                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
        },
        mounted() {
            this.handleQuery();
        },
    }
</script>
