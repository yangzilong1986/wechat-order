<template>

    <div>
          <Input v-model="search" @on-enter="searchAppList" @on-click="searchAppList" icon="ios-search" placeholder="搜索" style="width: 200px"></Input>
          <br/>
          <br/>
          <Button type="primary" size="small" @click="handleAddUser('add')" ><Icon type="ios-add" />添加人员</Button>
          <Button type="primary" size="small" @click="exportData(1)"><Icon type="ios-download-outline"></Icon> 导出数据</Button>
          <Button type="warning" size="small" @click="refresh"><Icon type="ios-aperture" /> 刷新</Button>
          <br>
          <br>
          <Table
            :columns="columns"
            border
            :data="data"
            size="small"
            ref="table">
          </Table>
          <br>
            <Page
              :total="page.total"
              :page-size="page.length"
              :page-size-opts="pageOptions"
              :current="page.start"
              size="small"
              @on-change="changePage"
              @on-page-size-change="changePageSize"
              show-total
              show-elevator
              show-sizer>
            </Page>
             <!-- add food -->
            <add-user
                :visible="addUserVisible"
                @close="closeAdd"
                :catList="catList"
                :data="UserData">
            </add-user>
    </div>
</template>

<script>
import { getUserList,getUserCatList,DelUser,UpdateUser } from '@/api/user'
import addUser from './components/userAdd.vue';

export default {
  components: { addUser },
  data () {
    return {
      search:'',
      showDetailFlag:false,
      detailTitle:'',
      pageOptions:[10, 20, 50, 100],
      currentStatus:'all',
      addUserVisible:false,
      page:{
          'total':0,
          'start':1,
          'length':10,
      },
      columns: [
                    {
                        "title": "Id",
                        "key": "id",
                        "fixed": "left",
                        "width": 80
                    },
                    {
                        "title": "照片",
                        "key": "img",
                        "width": 100,
                        render: (h, params) => {
                            return h('img', {
                                attrs: {
                                    src: params.row.img,
                                    style:'max-width: 70px;height: 50px;margin:5px;border-radius: 2px;'
                                }
                            },params.row.img)
                        }
                    },
                    {
                        "title": "姓名",
                        "key": "title",
                        "width": 100,
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'text',
                                        size: 'small'
                                    },
                                    on: {
                                        click: () => {
                                           this.showDetail(params.row.name)
                                        }
                                    }
                                }, params.row.title)
                            ]);
                        }
                    },
                    {
                        "title": "身份",
                        "key": "cat",
                        "width": 150
                    },
                    {
                        "title": "联系电话",
                        "key": "tel",
                        "width": 120,
                        "sortable": true
                    },
                    {
                        "title": "联系邮箱",
                        "key": "email",
                        "width": 120,
                        "sortable": true
                    },
                    {
                        "title": "状态",
                        "key": "status",
                        "width": 120,
                        "sortable": true,
                        render: (h, params) => {
                            return h('Tag', {
                                props: {
                                    color: params.row.status== 1 ? 'green' :'red'
                                }
                            },params.row.status == 1 ? '正常' : '已禁用')
                        }
                    },
                    {
                        "title": "创建时间",
                        "key": "creat_date",
                        "width": 180,
                        "sortable": true
                    },
                    {
                        "title": "备注",
                        "key": "remarks"
                    },
                    {
                        "title": "操作",
                        "key": "action",
                        'fixed': 'right',
                        "width": 200,
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'text',
                                        size: 'small'
                                    },
                                    on: {
                                        click: () => {
                                           this.handleAddUser(params.row)
                                        }
                                    }
                                }, '编辑'),
                                h('Button', {
                                    props: {
                                        type: 'text',
                                        size: 'small',
                                        disabled: params.row.status == 1 ? true : false 
                                    },
                                    on: {
                                        click: () => {
                                           this.$Modal.confirm({
                                                title: '状态修改',
                                                content: '确认要启用该人员么？',
                                                okText: '确认',
                                                cancelText: '取消',
                                                onOk:() => {
                                                    let text = "启用成功!";
                                                    this.handleUpdate(params.row,text)
                                                }
                                            });
                                        }
                                    }
                                }, '启用'),
                                h('Button', {
                                    props: {
                                        type: 'text',
                                        size: 'small',
                                        disabled: params.row.status == 1 ? false : true 
                                    },
                                    on: {
                                        click: () => {
                                           this.$Modal.confirm({
                                                title: '状态修改',
                                                content: '确认要禁用该人员么？',
                                                okText: '确认',
                                                cancelText: '取消',
                                                onOk:() => {
                                                    let text = "禁用成功!";
                                                    this.handleUpdate(params.row,text);
                                                }
                                            });
                                        }
                                    }
                                }, '禁用'),
                                h('Button', {
                                    props: {
                                        type: 'text',
                                        size: 'small'
                                    },
                                    on: {
                                        click: () => {
                                           this.$Modal.confirm({
                                                title: '删除',
                                                content: '确认要删除该人员么？',
                                                okText: '确认',
                                                cancelText: '取消',
                                                onOk:() => {
                                                    this.handleDel(params.row.id)
                                                }
                                            });
                                        }
                                    }
                                }, '删除')
                            ]);
                        }
                    }
                ],
      data: [],
      catList:[],
      UserData:null
    }
  },
  mounted(){
    this.getList(this.page.start,this.page.length,this.currentStatus);
    this.getCatList();
  },
  methods:{
    refresh(){
        this.search = '' ;
        this.page.start = 1 ;
        this.getList(this.page.start,this.page.length,this.currentStatus);
    },
    statusChange(res){
        this.currentStatus = res ;
        this.getList(this.page.start,this.page.length,this.currentStatus);
    },
    getCatList(){
      getUserCatList().then(res => {
            console.info("userCatList======>",res)
            this.catList = res ;
        })
    },
    getList(start,size,status){
      getUserList().then(res => {
            console.info("userList======>",res)
            this.data = res ;
            this.page.total = res.length;
        })
      },
      //页码切换
      changePage(res){
        this.page.start = res ;
        this.getList(res,this.page.length,this.currentStatus)
      },
      changePageSize(res){
        this.page.length = res ;
        this.getList(this.page.start,res,this.currentStatus)
      },
    exportData (type) {
        if (type === 1) {
            this.$refs.table.exportCsv({
                filename: 'The original data'
            });
        } else if (type === 2) {
            this.$refs.table.exportCsv({
                filename: 'Sorting and filtering data',
                original: false
            });
        } else if (type === 3) {
            this.$refs.table.exportCsv({
                filename: 'Custom data',
                columns: this.columns.filter((col, index) => index < 4),
                data: this.data.filter((data, index) => index < 4)
            });
        }
    },
    searchAppList(){
      console.log("搜索")
    },
    showDetail(name){
      this.detailTitle = name ;
      this.showDetailFlag = true ;
    },
    handleDel(params){
        DelUser(params).then(res => {
            this.$Message.success('删除成功!');
        })
    },
    handleUpdate(params,text){
        UpdateUser(params).then(res => {
            this.$Message.success(text);
        })
    },
    handleAddUser(res){
        this.addUserVisible = true ;

        if(res != 'add'){
            console.log(res)
            this.UserData = res ;
        }
    },
    closeAdd(){
        console.log("close")
        this.addUserVisible = false ;
        this.getList(this.page.start,this.page.length,this.currentStatus);
    }
  }
}
</script>
<style lang="scss" scoped>
.cl-tabs{
  margin-top:20px;
}
ul {
  list-style: none;
  .cl-binding{
    margin-top: 20px;
    color: #696969;
    span{
      display: inline-block;
      width: 100px;
      margin-right: 25px;
      font-weight: 600;
      text-align: right;
      color: #696969;
    }
  }
}
.cl-push-detail{
  display:flex;
  justify-content: space-around;
}
</style>

