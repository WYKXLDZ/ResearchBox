<template>
  <el-main style="position:absolute;height:100%;width:100%;top:0px;left:0px;">
    <el-header class="top_bar">
        <el-row style="margin:0px;"> 
            <!-- <el-col class="user_bar" :span="4">
                        <el-avatar src="https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png"></el-avatar>
            </el-col> -->
            <el-col class="title_bar" :span="24" ><i class="el-icon-paperclip"></i> Paper管理 </el-col>
        </el-row>
    </el-header>
    <el-row class="ops" :gutter="10">
        <el-button class="xs_hidden" type="primary" icon="el-icon-document-add" style="float: left;margin-bottom:10px;margin-right:20px;" v-on:click="show_add_paper=!show_add_paper">添加Paper</el-button>
        <el-card class="add_board" v-show="show_add_paper">
            <el-form label-position="top" label-width="80px" :model="add_form" :rules="rules">
                <el-form-item label="标题" prop="add_title">
                    <el-input v-model="add_form.title"></el-input>
                </el-form-item>
                <el-form-item label="第一作者" prop="add_author">
                    <el-input v-model="add_form.author1"></el-input>
                </el-form-item>
                <el-form-item label="发表时间">
                    <el-row>
                        <el-col :span="12" class="block">
                            <el-date-picker
                            v-model="add_form.Ptime"
                            type="year"
                            value-format="timestamp"
                            placeholder="选择年">
                            </el-date-picker>
                        </el-col>
                            <el-col :span="12" class="block">
                                <el-date-picker
                                v-model="add_form.Ptime"
                                type="month"
                                value-format="timestamp"
                                placeholder="选择月">
                                </el-date-picker>
                            </el-col>
                    </el-row>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="add_paper">立即添加</el-button>
                    <el-button v-on:click="show_add_paper=false">取消</el-button>
                </el-form-item>
            </el-form>
        </el-card>
        <el-col :span="6" :xs="12" style="line-height:40px;"><el-checkbox v-model="include_finished" @change="change_finished">已完成</el-checkbox></el-col>
        <el-col :span="6" :xs="12" style="line-height:40px;"><el-checkbox v-model="list_mode" @change="change_list_mode">列表模式</el-checkbox></el-col>
        <el-col :span="6" :xs="24" style="line-height:40px;">
            <el-input v-model="fast_id" placeholder="ID快速导航">
                <el-button slot="append" icon="el-icon-search" @click="ID_go"></el-button>
            </el-input>
        </el-col>
        
        <el-col :span="24" :xs="24" class="under_search_bar">
            <el-select style="width:calc(100%);" v-model="select_paper_lite" :change="handlePush(select_paper_lite)" clearable autocomplete default-first-option filterable placeholder="请搜索">
                <el-option 
                style="margin:5px"
                v-for="item in paper_lite_list"
                :key="item.id"
                :label="item.title"
                :value="item.id">
                <div class="paper_query_item" :style="{width:item.process + '%'}">
                    
                </div>
                {{item.title}} <span style="background:rgba(0,0,0,.1);padding:0px 10px;border-radius:5px;">{{item.id}}</span>
                </el-option>

            </el-select>
        </el-col>
    </el-row>
    <el-card v-if="pagenum==1" v-loading="paperRecents_loading" class="paper">
        <div class="analysis_title">最近编辑：</div>
        <el-row class="paper_recent"  v-for="paper in paperRecents" :key="paper.id">
        <div class="paper_recent_process" v-bind:style="{width:paper.process+'%'}"></div>
        <el-col class="paper_recent_title text_wrap" :span="16"><span :title="paper.id" v-on:click="$router.push('/papereditor/'+paper.id)">{{paper.title}}</span> <span style="background:rgba(0,0,0,.1);padding:0px 10px;border-radius:5px;font-size:10px;">{{paper.id}}</span></el-col>
        <el-col class="paper_recent_author text_wrap" :span="4"><i class='el-icon-user'></i> {{paper.author1}}</el-col>
        <el-col class="paper_recent_updatedAt" :span="4" >{{paper.updatedAt.getFullYear()}}-{{paper.updatedAt.getMonth()}}-{{paper.updatedAt.getDate()}} {{paper.updatedAt.getHours()}}:{{paper.updatedAt.getMinutes()}}</el-col>
        </el-row>
    </el-card>

    <el-card  :class="'paper'+(paper.process==100?' finished':'')" v-for="paper in paper_list" :key="paper.id" v-show="!list_mode && (paper.process!=100 || include_finished)">
        <el-row :gutter="10">
            <el-col :span="(paper.background!=null && paper.background!='')?8:0" :xs="24" style="position:relative;">
                <div v-show="paper.background!=null && paper.background!=''" class="paper_img" style="border-radius:5px;cursor:zoom-in;" @click="scale">
                <img :src="paper.background"/></div>
            </el-col>
            <el-col :span="(paper.background!=null && paper.background!='')?16:24"  :xs="24">
                <div  class="paper_head" style="cursor:pointer" v-on:click="$router.push('/papereditor/'+paper.id)">
            <span>{{paper.title}}</span> <span style="background:rgba(0,0,0,.1);padding:0px 10px;border-radius:5px;font-size:10px;">{{paper.id}}</span>
        </div>
        <div class="meeting">
            {{paper.meeting}}
        </div>
        <el-row>
            <el-col :span="12" class="ptime">
                <span style="font-size:10px;color:#999;">submitted:</span> {{new Date(parseInt(paper.Ptime)).getMonth()+1}},{{new Date(parseInt(paper.Ptime)).getFullYear()}} 
            </el-col>
            <el-col :span="12" style="text-align:right;">
                <el-dropdown @command="handleCommand">
                    <el-button type="primary" icon="el-icon-menu" class="paper_dropdown_btn" circle>
                    </el-button>
                    <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item v-bind:command="{'type':'d','params':paper.id}" >删除</el-dropdown-item>
                        <el-dropdown-item v-bind:command="{'type':'e','params':paper.id}" >编辑</el-dropdown-item>
                        <el-dropdown-item v-bind:command="{'type':'a','param1':paper.id,'param2':paper.title}" >添加至...</el-dropdown-item>
                    </el-dropdown-menu>
                    </el-dropdown>
            </el-col>
        </el-row>
        <!-- <div v-for="o in 4" :key="o" class="text item">
            {{'列表内容 ' + o }}
        </div> -->
        
        <el-row>
            <el-tag class='micro_tag'  v-for="tag in paper.tags" :key="tag">{{tag}}</el-tag>
            <!-- <el-col :span="24" >
                
            </el-col> -->
        </el-row>
        <el-row style="margin-top:10px;">
            <el-tag class='table_tag'  v-for="tag in paper.tables" :key="tag" v-show="now_tables[now_tables_map[tag]]">{{now_tables[now_tables_map[tag]]?now_tables[now_tables_map[tag]].title:''}}</el-tag>
            <!-- <el-col :span="24" >
                
            </el-col> -->
        </el-row>
            </el-col>
        </el-row>
        
        <div class="status_bar" v-bind:style="{width:paper.process + '%'}"></div>
    </el-card>
    <el-card class="paper"  v-show="list_mode">
        <el-row :gutter="4" :class="'paper_row'+(paper.process==100?' finished':'')" v-for="paper in paper_list" :key="paper.id"  v-show="list_mode && (paper.process!=100 || include_finished)">
            <div class="paper_recent_process" v-bind:style="{width:paper.process+'%'}"></div>
            <el-col :span="10" :xs="24" style="font-weight:bold;" ><span style="cursor:pointer" v-on:click="$router.push('/papereditor/'+paper.id)">{{paper.title}}</span></el-col>         
            <!-- <el-col :span="2"><i class='el-icon-user'></i> {{paper.author1}}</el-col>  -->
            <el-col :span="2"  style="font-size: 10px;"><span>{{paper.meeting}}</span></el-col>
            <el-col :span="8"  :xs="18" style="overflow:hidden;">
                <el-tag class='micro_tag'  v-for="tag in paper.tags" :key="tag">{{tag}}</el-tag>
            </el-col>
            <el-col :span="4" style="font-size: 10px;">{{new Date(parseInt(paper.Ptime)).getMonth()+1}},{{new Date(parseInt(paper.Ptime)).getFullYear()}}</el-col>
        </el-row>
    </el-card>
    <el-card class="add_to_table" v-show="show_add_to_table">
        <el-row>
            <el-col :span="24" style="text-align:center;font-size:1.2em;font-weight:bold;">添加至</el-col>
        </el-row>
        <el-row>
            <el-col :span="24" style="font-size:10px;margin-top:10px;">Paper: <strong>{{add_to_table.paper_title}}</strong></el-col>
        </el-row>
        <el-row>
            <el-col :span="24" style="font-size:10px;margin-top:10px;">Table: <strong>{{add_to_table.table_title}}</strong></el-col>
        </el-row>
        <el-row>
            <el-collapse style="margin:10px 0px;" accordion>
                <el-collapse-item title="选择需要添加至的Table">
                    <el-tag style="cursor:pointer;" v-for="table in now_tables" :key="table.id" v-on:click="add_to_table_table(table.title,table.id)">{{table.title}}</el-tag>
                </el-collapse-item>
            </el-collapse>
        </el-row>
        <el-row>
            <el-button type="primary" @click="add_to_table_fun">立即添加</el-button>
            <el-button v-on:click="show_add_to_table=false">取消</el-button>
        </el-row>
    </el-card>
    <el-row class="pagination">
        <el-col :span="24" class='pagination_board' >
            <el-pagination
            background
            :current-page="parseInt(pagenum)"
            :pager-count="5"
            layout="prev, pager, next"
            :page-size="papers_per_page"
            :total="paper_lite_list.length"
            @current-change="toPage">
            </el-pagination>
        </el-col>
    </el-row>
  </el-main>
</template>

<script>
const $ = require("jquery");
const config = require("../../utils/config");
const Loadding = require("../../utils/loadding");
const axios = require('axios');
const ColorThief = require('colorthief').default;


export default {
    inject:['reload'],
  name: 'login',
  props: ["pagenum"],
  data(){
    var checkNull = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('该项不能为空'));
        }else{
            callback();
        }
    }
    return {
        colorThief : new ColorThief(),
        list_mode:true,
        show_add_paper:false,
        show_add_to_table:false,
        include_finished:true,
        fast_id:null,
        paperRecents:[],
        paperRecents_loading:true,
        now_tables:[],
        now_tables_title:[],
        now_tables_map:{},
        add_to_table:{
            paper_title:"", 
            paper_id:"",
            table_title:"",
            table_id:"",
        },

        add_form:{
            title:"",
            author1:"",
            Ptime:"",
            PtimeYear:"",
            PtimeMon:""
        },
        paper_list:[],
        paper_lite_list:[],
        select_paper_lite:'',
        papers_per_page:config.papers_per_page,
        rules: {
          add_title: [
            { validator: checkNull, trigger: 'blur' }
          ],
          add_author: [         
            { validator: checkNull, trigger: 'blur' }
          ],

        }
    }
  },
  mounted:function(){
      var that =this;
      console.log(ColorThief)
      document.getElementsByTagName('title')[0].innerText = "Papers";
      that.include_finished = localStorage.getItem("RSB_paperManager_finished") == "true"?true:false;
      that.list_mode = localStorage.getItem("RSB_paperManager_list_mode") == "true"?true:false;
      var first_loadding = new Loadding();
    first_loadding.add_title("初始化");
    first_loadding.__init__();
    first_loadding.add_process(
        "拉取paper数据",
        async function(){
            var returndata = await axios.get(
                config.server_host + "/api/paper/by_page/"+that.pagenum
            );
            console.log(returndata)
            for(var i = 0; i< returndata.data.data.length;i++){
                if(returndata.data.data[i] ==null)continue;
                if(returndata.data.data[i]['tags'] != null){
                    returndata.data.data[i]['tags'] = returndata.data.data[i]['tags']==''?[]:returndata.data.data[i]['tags'].split(';');
                }else returndata.data.data[i]['tags']=[]
                if(returndata.data.data[i]['tables'] != null){
                    returndata.data.data[i]['tables'] = returndata.data.data[i]['tables']==''?[]:returndata.data.data[i]['tables'].split(';');
                }else returndata.data.data[i]['tables']=[]
                returndata.data.data[i]['Ptime'] = new Date(returndata.data.data[i]['Ptime']).getTime();
                that.paper_list.push(returndata.data.data[i]);
            }
        }
    );
    first_loadding.add_process(
        "拉取paper数据",
        async function(){
            var returndata = await axios.get(
                config.server_host + "/api/paper/by_ntime_query"
            );
                    console.log(returndata)
                        that.paper_lite_list=returndata.data.data;

        }
    );
    first_loadding.add_process(
        "拉取table数据",
        async function(){
            var returndata = await axios.get(
                config.server_host + "/api/plan/by_ntime"
            );
            returndata.data.data.forEach((i,idx)=>{
                that.now_tables.push({"id":i.id,"title":i.title})
                that.now_tables_title.push(i.title);
                that.now_tables_map[i.id] = idx;
            })
        }
    );
    if(this.pagenum==1)first_loadding.add_process(
        "拉取Recents数据",
        async function(){
            var returndata = await axios.get(
                config.server_host + "/api/analysis/paperrecents"
                )
            for(var i =0;i<returndata.data.data.length;i++){
                returndata.data.data[i]['updatedAt'] = new Date(returndata.data.data[i]['updatedAt']);
                that.paperRecents.push(returndata.data.data[i])
                }
            that.paperRecents_loading=false;
        }
    );
    first_loadding.start();
  },
  methods:{
      handlePush(paperid) {
        if(String(paperid)=='' || String(paperid)=='-1')return;
        this.$router.push('/papereditor/'+paperid)
    },
    toPage:function(pageidx){
        this.$router.push('/papermanager/'+pageidx);
        this.reload()
    },
      add_paper:function(){
          var that = this;
          console.log(that.add_form);
          var loadding = new Loadding();
            loadding.add_title("初始化");
            loadding.__init__();
            loadding.add_process(
                "添加Paper",
                async function(){
                await axios.post(
                    config.server_host + "/api/paper/add",
                    {"title":that.add_form.title,"author1":that.add_form.author1,"Ptime":that.add_form.Ptime}
                );
                that.show_add_paper=false;
                that.reload();
            })
        loadding.start();
      },
      del_paper:function(id){
          var that = this;
          var loadding = new Loadding();
        loadding.add_title("初始化");
        loadding.__init__();
        loadding.add_process(
        "删除Paper",
        async function(){
        await axios.post(
            config.server_host + "/api/paper/del",
            {"id":id}
        );
                that.reload();
        })
        loadding.start();
      },
      handleCommand:function(command){
          if(command.type=='e'){
              this.$router.push('/papereditor/'+command.params);
              return;
          }
          if(command.type=='d'){
              this.del_paper(command.params);
              return;
          }
          if(command.type=="a"){
              this.add_to_table_paper(command.param2,command.param1);
              this.show_add_to_table=true;
              return;
          }
      },
      add_to_table_table:function(title, id){
          this.add_to_table.table_title = title;
          this.add_to_table.table_id= id;
      },
      add_to_table_paper:function(title, id){
          this.add_to_table.paper_title = title;
          this.add_to_table.paper_id= id;
      },
      add_to_table_fun:function(){
          if(this.add_to_table.paper_id=="" || this.add_to_table.table_id == ""){
              this.$notify.error({
                title: '添加至...',
                message: '信息不全'
                });
                return;
          }
          var that = this;
          console.log(that.add_to_table)
          var loadding = new Loadding();
            loadding.add_title("初始化");
            loadding.__init__();
            loadding.add_process(
            "添加Paper到table",
            async function(){

            await axios.post(
                config.server_host + "/api/paper/add_paper_table",
                {"paperid":that.add_to_table.paper_id,"tableid":that.add_to_table.table_id}
            );
                    that.$notify({
                        title: '成功',
                        message: that.add_to_table.paper_title+"\nTO\n"+that.add_to_table.table_title
                        });
                    that.show_add_to_table = false;
            })
            loadding.start();
      },
      change_finished:function(changedValue){
          localStorage.setItem("RSB_paperManager_finished",String(changedValue));
      },
      change_list_mode:function(changedValue){
          localStorage.setItem("RSB_paperManager_list_mode",String(changedValue));
      },
      scale:function(event){
          var scale_ele = event.currentTarget;
          $(scale_ele).addClass("img_scale");
          var cover = $("<div style='position:fixed;top:0px;left:0px;height:100%;width:100%;z-index:100000001;cursor: zoom-out;'></div>");
          $("body").append(cover);
          cover.on("click",function(){
                $(scale_ele).removeClass("img_scale");
                cover.remove();
          })
      },
      ID_go:function(){
          this.paper_lite_list.forEach(ele=>{
              if(ele.id == this.fast_id){
                  this.$router.push('/papereditor/'+this.fast_id)
                  return
              }
          })
          this.$message('无该ID');
          
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" rel="stylesheet/scss">
.paper_row{
    // display: flex;
    width: 100%;
    padding: 5px 10px !important;
    line-height: 30px;
    box-sizing: unset !important;
    margin: 5px 0px;
    border-radius: 5px;
    transition: ease 0.5s;
    .el-col{
        overflow:hidden; //超出的文本隐藏
        text-overflow:ellipsis; //溢出用省略号显示
        white-space:nowrap; //溢出不换行
        z-index:1;
            position: relative;
                display: list-item;
    }
}
.paper_row.finished{
box-sizing: unset !important;
}
</style>
