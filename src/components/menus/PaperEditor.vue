<template>
  <el-main style="position:absolute;height:100%;width:100%;top:0px;left:0px;">
    <el-header class="top_bar">
    <el-row style="margin:0px;">
      <!-- <el-col class="user_bar" :span="4">
                        <el-avatar src="https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png"></el-avatar>
      </el-col>-->
      <el-col class="title_bar" :span="24">
        <i class="el-icon-paperclip"></i> Paper管理
      </el-col>
    </el-row>
    </el-header>
    <el-row class="ops">
      <el-button type="primary" icon="el-icon-back" v-on:click="$router.back(-1)" circle></el-button>
      <el-button type="primary" icon="el-icon-finished" v-on:click="upload_all()" circle></el-button>
      <el-button
        type="primary"
        :icon="edit_mode?'el-icon-reading':'el-icon-edit'"
        @click="edit_mode=!edit_mode"
        style="border-radius:10px;"
      >{{edit_mode?'Read Mode':'Edit Mode'}}</el-button>
      <el-tooltip class="item" effect="dark" :content="id" placement="bottom">
      <el-button type="primary" icon="el-icon-info" v-on:click="show_id()" circle></el-button>
      </el-tooltip>
    </el-row>
    <el-card class="paper paper_info">
      <el-row>
        <el-col :span="12" :xs="24" :md="12" class="paper_link">
          <el-link v-bind:href="edit_form.link" target="_blank">{{edit_form.link}}</el-link>
        </el-col>
        <el-col
          :span="12"
          :xs="0"
          :md="12"
          style="text-align:right;"
        >{{new Date(parseInt(edit_form.Ntime)).getFullYear()}} - {{new Date(parseInt(edit_form.Ntime)).getMonth() + 1}}</el-col>
      </el-row>
      <!-- <div class="status_bar" v-bind:style="{width:edit_form.process + '%'}"></div> -->
      <el-row>
        <el-col :span="24" class="paper_title">{{edit_form.title}}</el-col>
      </el-row>
      <el-row>
        <el-col
          :span="24"
          class="paper_meeting"
        >{{edit_form.meeting!=''?edit_form.meeting:''}} {{new Date(parseInt(edit_form.Ptime)).getFullYear()}}</el-col>
      </el-row>
      <el-row>
        <el-col
          :span="24"
          class="paper_author"
        >{{edit_form.author1}} {{edit_form.author2==""?'':'·'}} {{edit_form.author2}}</el-col>
      </el-row>
      <el-row>
        <el-col :span="24" class="paper_abstract">
          <span>{{less_abstract?edit_form.abstract.substring(0,edit_form.abstract.indexOf('.',400)+1):edit_form.abstract}}{{less_abstract?"...":""}}</span>
          <span
            style="cursor: pointer;font-weight: bold;margin: 0px 10px;border-bottom: 1px solid;"
            @click="less_abstract=!less_abstract;"
          >(show {{less_abstract?"more":"less"}})</span>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24" class="paper_cite">
          <span>cited:</span>
          {{edit_form.cite}}
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24" class="paper_tags">
          <el-tag
            :key="tag"
            v-for="tag in edit_form.tags"
            :closable="edit_mode"
            :disable-transitions="false"
            @close="handleClose(tag)"
          >{{tag}}</el-tag>
          <el-input
            class="input-new-tag"
            v-if="inputVisible"
            v-model="inputValue"
            ref="saveTagInput"
            size="small"
            @keyup.enter.native="handleInputConfirm"
            @blur="handleInputConfirm"
          ></el-input>
          <el-button
            v-if="edit_mode&&!inputVisible"
            class="button-new-tag"
            size="small"
            style="margin-left: 20px;"
            @click="showInput"
          >+ New Tag</el-button>
        </el-col>
      </el-row>
    </el-card>
    <el-card class="paper">
      <el-tabs v-model="activeTab" @tab-click="handleClickTab">
        <el-tab-pane name="first" v-if="edit_mode">
          <span slot="label">
            <i class="el-icon-document"></i> 基础信息
          </span>
          <el-card class="paper_edit" style="margin-top:10px;border:0px;box-shadow:0px 0px 0px;">
            <el-form ref="form" :model="edit_form" label-position="top" label-width="80px">
              <el-form-item label="阅读进度">
                <el-slider v-model="edit_form.process" :step="20" show-stops></el-slider>
              </el-form-item>
              <el-form-item label="标题">
                <el-input
                  placeholder="请输入内容"
                  v-model="edit_form.title"
                  clearable
                  class="paper_edit_title"
                ></el-input>
              </el-form-item>
              <el-form-item label="作者">
                <el-row :gutter="20">
                  <el-col :span="12">
                    <el-input
                      placeholder="请输入内容"
                      v-model="edit_form.author1"
                      clearable
                      class="paper_edit_author"
                    ></el-input>
                  </el-col>
                  <el-col :span="12">
                    <el-input
                      placeholder="请输入内容"
                      v-model="edit_form.author2"
                      clearable
                      class="paper_edit_author"
                    ></el-input>
                  </el-col>
                </el-row>
              </el-form-item>
              <el-form-item label="发表会议/期刊">
                <el-input
                  placeholder="请输入内容"
                  v-model="edit_form.meeting"
                  clearable
                  class="paper_edit_meeting"
                ></el-input>
              </el-form-item>
              <el-form-item label="发表时间">
                <el-row>
                  <el-col :span="12" class="block">
                    <el-date-picker
                      style="width: calc(100% - 10px);"
                      v-model="edit_form.Ptime"
                      type="year"
                      value-format="timestamp"
                      placeholder="选择年"
                    ></el-date-picker>
                  </el-col>
                  <el-col :span="12" class="block">
                    <el-date-picker
                      style="width: calc(100% - 10px);"
                      v-model="edit_form.Ptime"
                      type="month"
                      value-format="timestamp"
                      placeholder="选择月"
                    ></el-date-picker>
                  </el-col>
                </el-row>
              </el-form-item>
              <el-form-item label="引用量">
                <el-input
                  placeholder="请输入内容"
                  v-model="edit_form.cite"
                  clearable
                  class="paper_edit_cite"
                ></el-input>
              </el-form-item>
              <el-form-item label="链接">
                <el-input
                  placeholder="请输入内容"
                  v-model="edit_form.link"
                  clearable
                  class="paper_edit_link"
                ></el-input>
              </el-form-item>
              <el-form-item label="封面">
                <el-input
                  placeholder="请输入内容"
                  v-model="edit_form.background"
                  clearable
                  class="paper_edit_link"
                ></el-input>
              </el-form-item>
              <el-form-item label="摘要">
                <el-input
                  placeholder="请输入内容"
                  type="textarea"
                  v-model="edit_form.abstract"
                  clearable
                  class="paper_edit_link abstract"
                ></el-input>
              </el-form-item>
            </el-form>
          </el-card>
        </el-tab-pane>
        <el-tab-pane name="second">
          <span slot="label">
            <i :class="edit_mode?'el-icon-paperclip':'el-icon-edit-outline'"></i>
            {{full_editor?edit_form.title:edit_mode?'内容编辑':'Markdown'}}
          </span>
          <el-card
            style="margin-top:10px;border:0px;box-shadow:0px 0px 0px;"
            :class="full_editor?'full-screen':''"
          >
            <el-row class="editor_header">
              <el-col :span="24" :xs="0" :md="24" style="position: relative;">
                <el-button
                  style="float: right;padding: 10px;margin-right: 10px;"
                  type="primary"
                  icon="el-icon-upload"
                  v-on:click="upload_all()"
                  circle
                  v-if="edit_mode"
                ></el-button>
                <el-button
                  style="float: right;padding: 10px;margin-right: 10px;"
                  type="primary"
                  :icon="'el-icon-'+(show_md_editor?'check':'edit')"
                  circle
                  @click="change_show_md_editor"
                  v-if="edit_mode"
                ></el-button>
                <el-button
                  style="float: right;padding: 10px;"
                  type="primary"
                  :icon="'el-icon-'+(full_editor?'bottom-right':'top-left')"
                  circle
                  @click="full_editor=!full_editor;"
                ></el-button>
                <el-button
                  style="float: right;padding: 10px;"
                  type="primary"
                  icon="el-icon-picture"
                  circle
                  @click="push_pic_show = !push_pic_show"
                  v-if="edit_mode"
                ></el-button>
                <el-button
                  style="float: right;padding: 10px;"
                  type="primary"
                  icon="el-icon-plus"
                  circle
                  @click="show_add_to_table=true"
                  v-if="edit_mode"
                ></el-button>
                <el-card v-show="push_pic_show" id="push-pic-board" v-loading="push_pic.loadding">
                  <img :src="push_pic.base64" style="width:100%;" />
                  <p
                    v-show="push_pic.base64==''  && !push_pic.pushed"
                    style="text-align:center;"
                  >NO IMAGE!</p>
                  <p
                    v-show="push_pic.base64=='' && push_pic.pushed"
                    style="text-align:center;"
                  >COPY THE LINK!</p>
                  <el-input placeholder="照片名称" v-model="push_pic.name" v-show="!push_pic.pushed">
                    <el-button slot="append" icon="el-icon-upload" v-on:click="push_a_pic"></el-button>
                  </el-input>
                  <el-input v-model="push_pic.raw" v-show="push_pic.pushed"></el-input>
                </el-card>
                <el-card class="add_to_table" v-show="show_add_to_table">
                  <el-row>
                    <el-col
                      :span="24"
                      style="text-align:center;font-size:1.2em;font-weight:bold;"
                    >添加至</el-col>
                  </el-row>
                  <el-row>
                    <el-col :span="24" style="font-size:10px;margin-top:10px;">
                      Paper:
                      <strong>{{add_to_table.paper_title}}</strong>
                    </el-col>
                  </el-row>
                  <el-row>
                    <el-col :span="24" style="font-size:10px;margin-top:10px;">
                      Table:
                      <strong>{{add_to_table.table_title}}</strong>
                    </el-col>
                  </el-row>
                  <el-row>
                    <el-collapse style="margin:10px 0px;" accordion>
                      <el-collapse-item title="选择需要添加至的Table">
                        <el-tag
                          style="cursor:pointer;"
                          v-for="table in now_tables"
                          :key="table.id"
                          v-on:click="add_to_table_table(table.title,table.id)"
                        >{{table.title}}</el-tag>
                      </el-collapse-item>
                    </el-collapse>
                  </el-row>
                  <el-row>
                    <el-button type="primary" @click="add_to_table_fun">立即添加</el-button>
                    <el-button v-on:click="show_add_to_table=false">取消</el-button>
                  </el-row>
                </el-card>
              </el-col>
            </el-row>
            <el-row class="under-editor-header" :gutter="10">
              <el-col :span="12" style="max-width:500px;" :xs="24" :md="12">
                <el-slider v-model="md_size" :step="0.1" :min="1" :max="2" show-stops show-input></el-slider>
              </el-col>
            </el-row>
            <el-row :gutter="10">
              <el-col
                :span="edit_mode&&show_md_editor?12:24"
                :xs="24"
                :md="edit_mode&&show_md_editor?12:24"
              >
                <div
                  id="md_editor"
                  class="markdown-body"
                  :style="{fontSize:md_size+'em'}"
                  v-html="markdown.render(edit_form.md)"
                ></div>
              </el-col>
              <el-col
                :span="edit_mode&&show_md_editor?12:0"
                :xs="0"
                :md="edit_mode&&show_md_editor?12:0"
              >
                <el-input
                  style="box-shadow: 5px 5px 10px rgba(0,0,0,.1);margin:10px 0px;"
                  type="textarea"
                  autosize
                  placeholder="请输入内容"
                  v-model="edit_form.md"
                ></el-input>
              </el-col>
            </el-row>
            <el-row>
              <el-col>
                <!-- <el-input
                    type="textarea"
                    autosize
                    placeholder="请输入内容"
                    v-model="edit_form.md">
                </el-input>-->
              </el-col>
            </el-row>
          </el-card>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  </el-main>
</template>

<script>
const $ = require("jquery");
const config = require("../../utils/config");
const Loadding = require("../../utils/loadding");
import markdownIt from "markdown-it";
import markdownItLatex from "markdown-it-latex";
import "markdown-it-latex/dist/index.css";
import "../../assets/github-markdown.min.css";
const axios = require("axios");

const md = markdownIt();
md.use(require("markdown-it-highlightjs"), { inline: true });
md.use(markdownItLatex);

export default {
  name: "login",
  props: ["id"],
  data() {
    var checkNull = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("该项不能为空"));
      } else {
        callback();
      }
    };
    return {
      markdown: md,
      edit_mode: true,
      show_md_editor: true,
      show_add_paper: false,
      full_editor: false,
      push_pic_show: false,
      show_add_to_table: false,
      activeTab: "first",
      now_tables:[],
      now_tables_title:[],
      add_to_table:{
            paper_title:"", 
            paper_id:"",
            table_title:"",
            table_id:"",
        },
      push_pic: {
        name: "",
        base64: "",
        raw: "",
        loadding: false
      },
      md_size: 1,
      edit_form: {
        title: "",
        author1: "",
        author2: "",
        cite: "",
        meeting: "",
        tags: [],
        process: 0,
        link: "",
        Ptime: "",
        Ntime: "",
        md: "",
        abstract: "",
        background: ""
      },
      less_abstract: true,
      inputVisible: false,
      inputValue: "",
      paper_list: [],
      rules: {
        add_title: [{ validator: checkNull, trigger: "blur" }],
        add_author: [{ validator: checkNull, trigger: "blur" }]
      },
      github_info: {
        username: "",
        repos: "",
        token: ""
      }
    };
  },
  mounted: function() {
    var hljs_link = document.createElement("link");
    document.head.appendChild(hljs_link);

    hljs_link.outerHTML =
      "<link rel='stylesheet' href='//cdn.jsdelivr.net/gh/highlightjs/cdn-release@10.2.1/build/styles/default.min.css'>";
    var that = this;
    var first_loadding = new Loadding();
    first_loadding.add_title("初始化");
    first_loadding.__init__();
    var _show_md_editor = localStorage.getItem("RSB_show_md_editor");
    if (_show_md_editor) {
      this.show_md_editor = _show_md_editor == "false" ? false : true;
    } else {
      localStorage.setItem("RSB_show_md_editor", this.show_md_editor);
    }
    first_loadding.add_process("拉取数据", async function() {
      var returndata = await axios.get(
        config.server_host + "/api/paper/fetchone/" + that.id
      );
      var data = returndata.data.data[0];
      var s = [
        "title",
        "author1",
        "author2",
        "cite",
        "Ptime",
        "Ntime",
        "meeting",
        "link",
        "background"
      ];
      that.source_title = data["title"];
      for (var i = 0; i < s.length; i++) that.edit_form[s[i]] = data[s[i]];
      if (data["tags"] != null) {
        that.edit_form["tags"] =
          data["tags"] == "" ? [] : data["tags"].split(";");
        console.log(data["tags"]);
      } else that.edit_form["tags"] = [];
      that.edit_form["process"] = parseInt(data["process"]);
      that.edit_form["md"] = data["md"] == null ? "" : data["md"];
      that.edit_form["abstract"] =
        data["abstract"] == null ? "" : data["abstract"];
      that.edit_form["Ptime"] = new Date(data["Ptime"]).getTime();
      that.edit_form["Ntime"] = new Date(data["Ntime"]).getTime();
      document.getElementsByTagName("title")[0].innerText =
        that.edit_form["title"];
    });
    first_loadding.add_process("拉取table数据", async function(){
            var returndata = await axios.get(
                config.server_host + "/api/plan/by_ntime"
            );
                    for(var i in returndata.data.data){
                        that.now_tables.push({"id":returndata.data.data[i].id,"title":returndata.data.data[i].title})
                        that.now_tables_title.push(returndata.data.data[i].title);
                    }
                    that.add_to_table.paper_title = that.edit_form.title;
      that.add_to_table.paper_id = that.id;
        })
    first_loadding.add_process("添加粘贴事件", async function() {
      that.add_paste_event();
    });
    first_loadding.add_process("拉取Github数据", async function() {
      var returndata = await axios.get(config.server_host + "/api/user/info");
      that.user_info = returndata.data.data;
      if (that.user_info.github_info == null) {
        console.log("no github_info was settled.");
      } else {
        that.user_info.github_info = JSON.parse(that.user_info.github_info);
        that.github_info.username = that.user_info.github_info.username;
        that.github_info.repos = that.user_info.github_info.repos;
        that.github_info.token = that.user_info.github_info.token;
      }
    });
    first_loadding.start();
  },
  methods: {
    open: function(link) {
      window.open(link);
    },
    upload_all: function() {
      var that = this;
      var loadding = new Loadding();
      loadding.add_title("初始化");
      loadding.__init__();
      loadding.add_process("更新数据", async function() {
        var form = {};
        for (var key in that.edit_form) form[key] = that.edit_form[key];
        form["tags"] = form["tags"].join(";");
        await axios.post(config.server_host + "/api/paper/uploadone", {
          id: that.id,
          new_paper: form
        });
      });
      loadding.start();
    },
    handleClose: function(tag) {
      this.edit_form.tags.splice(this.edit_form.tags.indexOf(tag), 1);
    },

    showInput: function() {
      this.inputVisible = true;
      this.$nextTick(_ => {
        console.log(_);
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    handleClickTab: function() {},
    handleInputConfirm: function() {
      let inputValue = this.inputValue.toLowerCase();
      if (inputValue) {
        this.edit_form.tags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = "";
    },
    change_show_md_editor: function() {
      this.show_md_editor = !this.show_md_editor;
      localStorage.setItem("RSB_show_md_editor", this.show_md_editor);
    },
    add_paste_event: function() {
      var that = this;
      const stopwords = ",.'\":; ";
      document.addEventListener("paste", function(event) {
        if (!that.push_pic_show) {
          return;
        }

        var items = event.clipboardData && event.clipboardData.items;
        var file = null;
        var reader = new FileReader();
        if (items && items.length) {
          // 检索剪切板items
          for (var i = 0; i < items.length; i++) {
            if (items[i].type.indexOf("image") !== -1) {
              file = items[i].getAsFile();
              break;
            }
          }
        }
        var name = that.edit_form.title.substring(0, 15).toLowerCase();
        for (var w = 0; w < stopwords.length; w++) {
          name = name.split(stopwords[w]).join("_");
        }
        name = that.id + "_" + name + "_1.png";
        reader.onload = function(event) {
          that.push_pic.base64 = event.target.result;
          that.push_pic.name = name;
          that.push_pic.pushed = false;
        };
        if (file) {
          reader.readAsDataURL(file);
        }
      });
    },
    push_a_pic: function() {
      var that = this;
      that.push_pic.loadding = true;
      if (
        that.github_info.username == "" ||
        that.github_info.repos == "" ||
        that.github_info.token == ""
      ) {
        that.$notify({
          title: " 失败",
          message: "Github 信息有误"
        });
        return;
      }
      $.ajax({
        url:
          "https://gitee.com/api/v5/repos/" +
          that.github_info.username +
          "/" +
          that.github_info.repos +
          "/contents/" +
          that.push_pic.name,
        type: "GET",
        contentType: "application/json",
        dataType: "json",
        headers: { Authorization: "token " + that.github_info.token },
        success: function(returndata) {
          if (returndata.length == 0) {
            // put a new pic to gitee repos
            $.ajax({
              url:
                "https://gitee.com/api/v5/repos/" +
                that.github_info.username +
                "/" +
                that.github_info.repos +
                "/contents/" +
                that.push_pic.name,
              type: "POST",
              contentType: "application/json",
              dataType: "json",
              data: JSON.stringify({
                access_token: that.github_info.token,
                owner: that.github_info.username,
                repo: that.github_info.repos,
                path: that.push_pic.name,
                message: "upload a pic named " + that.push_pic.name,
                content: that.push_pic.base64.substring(22)
              }),
              headers: { Authorization: "token " + that.github_info.token },
              success: function(returndata) {
                // console.log(returndata)
                that.$notify({
                  title: "成功",
                  message: that.push_pic.name + " 上传成功！"
                });
                that.push_pic.raw = returndata.content.download_url;
                that.push_pic.base64 = "";
                that.push_pic.pushed = true;
                that.push_pic.loadding = false;
              },
              error: function(returndata) {
                that.$notify({
                  title: " 失败",
                  message: returndata.responseJSON.message
                });
                that.push_pic.loadding = false;
              }
            });
          } else {
            that
              .$alert("是否替换" + that.push_pic.name, "替换", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
              })
              .then(() => {
                $.ajax({
                  url:
                    "https://gitee.com/api/v5/repos/" +
                    that.github_info.username +
                    "/" +
                    that.github_info.repos +
                    "/contents/" +
                    that.push_pic.name,
                  type: "PUT",
                  contentType: "application/json",
                  dataType: "json",
                  data: JSON.stringify({
                    access_token: that.github_info.token,
                    owner: that.github_info.username,
                    repo: that.github_info.repos,
                    path: that.push_pic.name,
                    message: "upload a pic named " + that.push_pic.name,
                    content: that.push_pic.base64.substring(22),
                    sha: returndata.sha
                  }),
                  headers: { Authorization: "token " + that.github_info.token },
                  success: function(returndata) {
                    // console.log(returndata)
                    that.$notify({
                      title: "成功",
                      message: that.push_pic.name + " 替换成功！"
                    });
                    that.push_pic.raw = returndata.content.download_url;
                    that.push_pic.base64 = "";
                    that.push_pic.pushed = true;
                    that.push_pic.loadding = false;
                  },
                  error: function(returndata) {
                    that.$notify({
                      title: " 失败",
                      message: returndata.responseJSON.message
                    });
                    that.push_pic.loadding = false;
                  }
                });
              })
              .catch(() => {
                that.$notify({
                  title: "取消替换",
                  message: ""
                });
                that.push_pic.loadding = false;
              });
          }
        },
        error: function() {
          $.ajax({
            url:
              "https://gitee.com/api/v5/repos/" +
              that.github_info.username +
              "/" +
              that.github_info.repos +
              "/contents/" +
              that.push_pic.name,
            type: "PUT",
            contentType: "application/json",
            dataType: "json",
            data: JSON.stringify({
              message: "upload a pic named " + that.push_pic.name,
              content: that.push_pic.base64.substring(22)
            }),
            headers: { Authorization: "token " + that.github_info.token },
            success: function(returndata) {
              // console.log(returndata)
              that.$notify({
                title: "成功",
                message: that.push_pic.name + " 上传成功！"
              });
              that.push_pic.raw = returndata.content.download_url;
              that.push_pic.base64 = "";
              that.push_pic.pushed = true;
              that.push_pic.loadding = false;
            },
            error: function(returndata) {
              that.$notify({
                title: " 失败",
                message: returndata.responseJSON.message
              });
              that.push_pic.loadding = false;
            }
          });
        }
      });
    },
    add_to_table_table:function(title, id){
          this.add_to_table.table_title = title;
          this.add_to_table.table_id= id;
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
            "删除Paper",
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
      show_id:function(){
        // id_ = this.id + this.edit_form.title.split(" ").join("_")
        this.$alert("<input style='width:100%;' type=input value='"+this.id +"_"+ this.edit_form.title.split(" ").join("_")+"'/>", '编号', {
          // confirmButtonText: '确定',
          dangerouslyUseHTMLString: true,
          callback: () => {
            // this.$message({
            //   type: 'info',
            //   message: `action: ${ action }`
            // });
          }
        });
      }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" rel="stylesheet/scss" scoped>
</style>
