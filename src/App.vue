<template>
  <div id="app">
    <header>图片管理</header>
    <section>
      <div class="operate">
        <button type="button" class="btn btn-primary" @click="isUploadModalShow=true">上传图片</button>
      </div>
      <div class="food-list">
        <ul>
          <li v-for="svrPic in svrPicList">
            <img :title="svrPic.name" v-bind:src="'/static/'+svrPic.src" alt="">
            <div class="del-pic" @click="modalConfirm=true">删除</div>
          </li>
        </ul>
      </div>
    </section>
    <div class="overlay" v-show='isUploadModalShow'></div>
    <div class="upload-modal" v-show='isUploadModalShow'>
      <header>上传图片</header>
      <div class="body">
        <div class="tips">请选择上传的图片<span>（上传图片必须小于200kb，限png，jpg格式）</span></div>
        <div class="panel">
          <div class="panel-header">要上传的图片<span>({{cliPicList.length}}/10)</span></div>
          <ul class="pic-list" v-show="cliPicList.length">
            <li v-for="cliPic in cliPicList">
              <span class="title" :title="cliPic.name">{{cliPic.name}}</span>
              <span v-if="cliPic.status=='0'" title="待上传" class="status wait-upload">待上传</span>
              <span v-else-if="cliPic.status=='1'" title="正在上传" class="status uploading">正在上传</span>
              <span v-else-if="cliPic.status=='2'" title="上传完成" class="status upload-finished">上传完成</span>
              <span v-else="cliPic.status=='3'" title="超过大小，无法上传" class="status too-big">超过大小，无法上传</span>
            </li>
          </ul>
          <div class="panel-footer" v-show="isUploading">
            正在上传1/3
          </div>
        </div>
        <div class="btn-group" v-show="!isUploading&&!isUploadFinished">
          <button v-show="cliPicList.length<10" type="button" class="select-pic btn"
                  v-bind:class="{'btn-primary':cliPicList.length==0,'btn-default':cliPicList.length!=0}"
                  @click="selectPic">
            选择图片
          </button>
          <button type="button" v-show="cliPicList.length>0" class="upload btn btn-primary"
                  @click="upload">上传
          </button>
        </div>
        <div class="upload-result" v-show="isUploadFinished">上传完成</div>
      </div>
      <footer><span @click="closeUploadModal">取消</span></footer>
    </div>
    <modal v-bind:mdShow="modalConfirm" @close="closeModal">
      <p slot="message">你确认要删除这张图片吗？</p>
      <div slot="btnGroup" class="btn-group">
        <button class="btn btn-primary" href="javascript:;" @click="delPic">确认</button>
        <button class="btn btn-primary" href="javascript:;" @click="modalConfirm=false">取消</button>
      </div>
    </modal>
    <modal v-bind:mdShow="isWarnShow" v-on:close="closeModal">
      <p slot="message">
        {{modalTips}}
      </p>
      <div slot="btnGroup" class="btn-group">
        <button class="btn btn-primary" href="javascript:void(0)" @click="isWarnShow=false">关闭</button>
      </div>
    </modal>
  </div>
</template>

<script>
  import './assets/base.css'
  import axios from 'axios'
  import modal from './components/modal.vue'
  import svrPicListJson from '../mock/svrPicList.json'

  export default {
    name: 'app',
    data() {
      return {
        hello: 'hello',
        svrPicList: [],//服务器图片
        modalConfirm: false,
        isUploadModalShow: false,//呈现上传弹窗
        isWarnShow: false,
        isUploadFinished: false,//呈现上传结果
        max: 200 * 1024, //图片最大字节
        cliPicList: [],//客户端图片
        isUploading: false,
        modalTips: ''
      }
    },
    components: {
      modal
    },
    mounted() {
      this.getSvrPicList();
    },
    methods: {
      //获取服务器图片列表
      getSvrPicList() {
        this.svrPicList = svrPicListJson.result;
      },
      delPic() {
        this.modalConfirm = true;
      },
      closeModal() {
        this.modalConfirm = false;
        this.isWarnShow = false;
      },
      //选择图片
      selectPic() {
        let that = this;
        let inputFile = document.createElement('input');
        inputFile.type = 'file';
        inputFile.multiple = 'multiple';
        inputFile.accept = '.png,.jpg';
        let names = [];
        that.cliPicList.concat(that.svrPicList).forEach((item) => {
          names.push(item.name);
        });
        inputFile.onchange = function (event) {
          let files = event.target.files;
          if (files.length) {
            for (let i = 0; i < files.length; i++) {
              //status:'0':待上传，'1'：正在上传，'2'：已上传，'3'：超过大小，无法上传
              let status = '0';
              if (files[i].size > that.max) {
                status = '3';
              }
              let pic = {
                name: files[i].name,
                status: status,
              };
              if (names.indexOf(pic.name) != -1) {
                that.isWarnShow = true;
                that.modalTips = '同名文件已存在!';
                continue;
              }
              that.cliPicList.push(pic);
              if (files.length > 1 && that.cliPicList.length == 10) {
                that.isWarnShow = true;
                that.modalTips = '一次最多只能上传10张图片!';
                break;
              }
            }
          }
        }
        inputFile.click();
        inputFile = null;
      },
      //上传
      upload() {
        this.isUploading = true;
        this.cliPicList.forEach((item) => {
          if (item.status != '3') {
            item.status = '1'
          }
        });
        setTimeout(() => {
          this.isUploading = false;
          this.isUploadFinished = true;
          this.cliPicList.forEach((item) => {
            if (item.status != '3') {
              item.status = '2'
            }
          });
        }, 1000);
      },
      closeUploadModal() {
        this.isUploadModalShow = false;
        this.isUploadFinished = false;
        this.cliPicList = []
      }
    }
  }
</script>

<style>
  /*#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }*/
</style>
