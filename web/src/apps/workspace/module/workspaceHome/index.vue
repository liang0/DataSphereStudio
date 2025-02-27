<template>
  <div class="page-bgc">
    <div class="page-bgc-header">
      <div class="header-info">
        <h1>{{$t('message.workspace.home.welcomeAppShop', {text: workspaceData.name})}}</h1>
        <p>{{workspaceData.description}}</p>
      </div>
    </div>
    <div class="workspace-main">
      <Card class="right">
        <div class="nodata-tips" v-if="!adminApps.title">{{$t('message.workspace.home.tips')}}</div>
        <h3 v-if="adminApps.title" class="item-header">
          <span>{{adminApps.title}}</span>
        </h3>
        <div v-if="adminApps.title" class="app-list">
          <div v-for="item in adminApps.appInstances" :key="item.title" class="app-item-wrap" @click="navTo(item, item.accessButtonUrl)">
            <SvgIcon class="app-icon" :icon-class="iconSplit(item.icon)[0]" :color="iconSplit(item.icon)[1]"/>
            <span class="label">{{item.accessButton}}</span>
          </div>
        </div>
      </Card>
    </div>

    <div class="app-list-main">
      <div class="app-list-tabs" :class="{hideBar: search}">
        <div class="nodata-tips" v-if="tabsApplication.length===0">{{$t('message.workspace.home.tips')}}</div>
        <Tabs v-else active-key=0>
          <Tab-pane v-for="(type, index) in tabsApplication" :label="type.title" :key="index">
            <div class="pane-wrap">
              <Card v-for="item in tabsApplication[index].appInstances" :key="item.name" class="pane-item">
                <div class="app-entrance">
                  <div class="app-title-wrap">

                    <div class="app-title">
                      <SvgIcon class="app-icon title-sub" :icon-class="iconSplit(item.icon)[0]" :color="iconSplit(item.icon)[1]"/>
                      <span class="label title-sub">{{item.title}}</span>
                      <Tag class="app-tag title-sub" v-for="tag in (item.labels ? item.labels.split(','):[])" :key="tag">{{tag}}</Tag>
                    </div>


                    <p>{{item.description}}</p>
                  </div>

                  <div v-if="item.active" class="app-status-wrap-active">
                    <SvgIcon class="status-icon" icon-class="fi-radio-on2"/>
                    <span>{{$t('message.workspace.home.running')}}</span>
                  </div>
                  <div v-else class="app-status-wrap-disable">
                    <SvgIcon class="status-icon" icon-class="fi-radio-on2"/>
                    <span>{{$t('message.workspace.home.stop')}}</span>
                  </div>

                </div>

                <div v-if="item.active" class="button-wrap">
                  <Button class="entrace-btn" size="small" type="primary" @click="linkTo(item, item.accessButtonUrl)">{{item.accessButton}}</Button>
                  <Button class="entrace-btn" size="small" @click="navTo(item, item.manualButtonUrl)">{{item.manualButton}}</Button>
                </div>
                <div v-else class="button-wrap">
                  <Button class="entrace-btn" size="small" type="primary" disabled @click="navTo(item, item.accessButtonUrl)">{{item.accessButton}}</Button>
                  <Button class="entrace-btn" size="small" @click="navTo(item, item.manualButtonUrl)">{{item.manualButton}}</Button>
                </div>
              </Card>

            </div>

          </Tab-pane>
        </Tabs>

        <div class="input-wrap">
          <Input icon="ios-search" :placeholder="$t('message.workspace.home.searchPlaceholder')" style="width: 200px" @on-change="onSearch" />
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import api from '@/common/service/api';
import mixin from '@/common/service/mixin';
import util from '@/common/util';
import { GetWorkspaceData, GetWorkspaceApplications } from '@/common/service/apiCommonMethod.js';
export default {
  created(){
    this.workspaceId = this.$route.query.workspaceId;
    this.init();
  },


  watch: {
    $route() {
      this.workspaceId= this.$route.query.workspaceId; //获取传来的参数
      this.init();
    }
  },
  data: function() {
    return {
      workspaceId: null,
      workspaceData: {
        name: "",
        description: ""
      },
      adminApps: {},
      show: false,
      applications: [],
      searchResult: [],
      addAppLoading: false,
      rules: {
        selectApp: [
          { required: true, message: this.$t('message.workspace.home.selectApp'), trigger: 'change' },
        ],
        selectType: [
          { required: true, message: this.$t('message.workspace.home.selectType'), trigger: 'change' },
        ],
      },
      search: false,
    };
  },
  mixins: [mixin],
  methods: {
    iconSplit(icon){
      if(icon){
        return icon.split('|')
      }
      return ['','']
    },
    init(){
      GetWorkspaceData(this.workspaceId).then(data=>{
        this.workspaceData = data.workspace;
      })
      api.fetch(`${this.$API_PATH.WORKSPACE_PATH}workspaces/${this.workspaceId}/managements`, 'get').then(data=>{
        this.adminApps = data.managements[0] ? data.managements[0]: {};
      })

      GetWorkspaceApplications(this.workspaceId).then(data=>{
        this.applications = data.applications || [];
      })
    },
    flatApps(){
      const arr = [];
      this.applications.forEach(item=>{
        arr.push(...item.appInstances)
      })
      return arr;
    },

    onSearch(event){
      const value = event.target.value;
      if(value){
        const arr = this.flatApps();
        this.searchResult = arr.filter(item=>{
          if(item.title.indexOf(value)!==-1 || item.labels.indexOf(value)!==-1){
            return true;
          }
          return false;
        })
        this.searchResult = [{title: 'result', appInstances: [...this.searchResult]}];
        this.search = true;
      }else{
        this.search = false;
      }
    },
    linkTo(item){
      this.gotoCommonIframe(item.name, {workspaceId: this.workspaceId})
    },
    navTo(item, path){
      if(path){
        if(path.startsWith('http')){
          util.windowOpen(path);
        }else {
          this.$router.push({path: path, query: Object.assign({}, this.$route.query)});
        }
      }else {
        console.warn('path error', path);
      }
    }
  },

  computed: {

    tabsApplication: function(){
      if(this.search){
        return this.searchResult;
      }
      return this.applications;
    }
  }
}
</script>
<style lang="scss" scoped src="../../assets/styles/workspace.scss"></style>

<style lang="scss" scoped>
@import '@/common/style/variables.scss';
  .hideBar > .ivu-tabs > .ivu-tabs-bar {
      visibility: hidden;
      padding-top: 35px;
    }

    .ivu-tabs > .ivu-tabs-bar > .ivu-tabs-nav-container{
      display: block;
    }
    .hideBar > .ivu-tabs > .ivu-tabs-bar > .ivu-tabs-nav-container {
      display: none;
    }
    .page-bgc-header .header-info h1 {
      font-size: $font-size-large;
    }
    .page-bgc-header .header-info p {
      /* text-indent: 2rem; */
      font-size: $font-size-small;
    }
  .app-tag > .ivu-tag-text {
    color: #fff;
    border-radius: 2px;
    border-radius: 2px;
  }
</style>