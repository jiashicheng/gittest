<template>
  <div class="basicInformation"  v-if="dataInfo">
    <div class="split-info-box">
      <h3>服务信息：</h3>
      <div class="flex-box">
        <p><span>服务状态：</span><span v-if="dataInfo.deviceStatus == 2">服务中</span></p>
        <p><span>安装位置：</span><span>{{dataInfo.shipBook}}</span></p>
        <p><span>建站人员：</span><span>{{dataInfo.buildPerson}}</span></p>
        <p><span>安装人员：</span><span>{{dataInfo.installPerson}}</span></p>
        <p><span>解除服务人员：</span><span>{{dataInfo.removePerson}}</span></p>
        <p><span>合伙人：</span><span>{{dataInfo.partner}}</span></p>
        <p><span>IP电话号码：</span><span>{{dataInfo.ipPhone}}</span></p>
        <p><span>其他联系方式：</span><span>{{dataInfo.otherContact}}</span></p>
        <p><span>建站时间：</span><span>{{dataInfo.createTime}}</span></p>
        <!-- <p><span>x投入服务时间：</span><span>{{dataInfo.serviceTime}}</span></p> -->
        <!-- <p><span>x解除服务时间：</span><span>{{dataInfo.removeTime}}</span></p> -->
      </div>
    </div>
    <div class="split-info-box">
      <h3>天线信息：</h3>
      <div class="flex-box">
        <p><span>天线状态：</span><span>{{dataInfo.antennaStatus}}</span></p>
        <p><span>天线厂家：</span><span>{{dataInfo.antennaProducer}}</span></p>
        <p><span>天线编号：</span><span>{{dataInfo.antennaNumber}}</span></p>
      </div>
    </div>
    <div class="split-info-box no-border">
      <h3>盒子基础信息：</h3>
      <div class="flex-box">
        <p><span>基础服务在线状态：</span><span>{{dataInfo.onlineStatus==1 ? '在线' : '离线'}}</span></p>
        <p><span>运维服务在线状态：</span><span>{{dataInfo.opsOnline == 1 ? '在线' : '离线'}}</span></p>
        <p><span>盒子MAC地址：</span><span>{{dataInfo.mac}}</span></p>
        <p><span>盒子编号：</span><span>{{dataInfo.sn}}</span></p>
        <p><span>硬件版本：</span><span>{{dataInfo.hardwareVersion}}</span></p>
      </div>
    </div>

    我在这里增加了1.0的内容
    <div class="software-version" v-if="isShowVer">
      <p class="version-title"><span>软件版本：</span></p>
      <el-table :data="versionData" stripexborder size="small">
        <!-- <el-table-column prop="appId" label="软件模块编号" min-width="120"></el-table-column> -->
        <!-- <el-table-column prop="appName" label="软件模块名称" min-width="180"></el-table-column> -->
        <el-table-column prop="version" label="版本号" min-width="180">
          <template slot-scope="scope">
            <span>{{scope.row.type === 2 && scope.row.versionDesc ? scope.row.versionDesc.targetVersion : scope.row.version}}</span>
          </template>
        </el-table-column>
      </el-table>
    </div>

    我修改了1.0的bug
  </div>
</template>

<script>
// import { FisherEdgeFindBySn, GetAppInfo } from '../../../../utils/urlApi.js';
// import { setTime } from '../../../../utils/public.js';
// export default {
  name: "BasicInformation",
  components: {},
  props: {
    msg: String,
    sddd: '我增加了2.0的需求在这里呢',
  },
  data() {
    return {
      dataInfo: null,
      sn: '',
      isShowVer: true,
      versionData: [],
    };
  },
  created: function() {},
  computed: {},
  watch: {
    routerInfo: {
      handler(newValue, oldValue) {},
      deep: true
    }
  },
  mounted: function() {
    this.sn = this.$route.query.sn;
    this.getDataInfo();
    this.getVersion();
  },
  methods: {
    showSoftwareVersion () {
      this.getVersion();
      this.isShowVer = !this.isShowVer;
    },
    getDataInfo() {
      let params = {
        sn: this.sn
      };
      this.$showLoading();
      this.$http
        .get(FisherEdgeFindBySn, params)
        .then(response => {
          this.$hideLoading();
          if (response.success) {
            let dataInfo = response.data;
            if (dataInfo.createTime) {
              dataInfo.createTime = setTime(dataInfo.createTime);
            } else {
              dataInfo.createTime = '--';
            }
            this.dataInfo = dataInfo;
            this.$emit('renderShipName', dataInfo.shipName);
          } else {
            this.$message.error(response.errorMsg);
          }
        })
        .catch(err => {});
    },
    getVersion() {
      let params = {
        sn: this.sn
      };
      this.$http
        .get(GetAppInfo, params)
        .then(response => {
          // response = {"success":true,"data":[{"appId":1,"version":"2.0.0.7","appName":"基础服务","type":0,"versionDesc":null},{"appId":2,"version":"2.0.0.2","appName":"服务治理","type":1,"versionDesc":null},
          // {"appId":2,"version":"2.0.0.2","appName":"服务治理","type":1,"versionDesc":null},
          // {"appId":2,"version":"2.0.0.2","appName":"服务治理","type":2,"versionDesc":{"targetVersion": '111.222'}},
          // {"appId":2,"version":"2.0.0.2","appName":"服务治理22","type":1,"versionDesc":null},
          // {"appId":2,"version":"2.0.0.2","appName":"服务治理33","type":2,"versionDesc":{"targetVersion": '111.222.333'}},
          // {"appId":6,"version":"2.0.0.8","appName":"抖音服务","type":1,"versionDesc":null},{"appId":7,"version":"2.0.0.2","appName":"应用市场服务","type":1,"versionDesc":null}],"errorCode":null,"errorMsg":null,"message":{}};
          if (response.success) {
            let tempArr = [];
            let tempObj = {};
            response.data.forEach((item, index, array) => {
              if(item.type === 2) {
                let targetVersion = item.versionDesc['targetVersion'];
                if(tempObj[item.appId]) {
                  tempObj[item.appId] = tempObj[item.appId] + ',' + targetVersion;
                } else {
                  tempArr.push(item);
                  let targetVersion = item.versionDesc['targetVersion'];
                  tempObj[item.appId] = targetVersion;
                }
              } else {
                tempArr.push(item);
              }
            })
            let tempArr2 = tempArr.map((item) => {
              if(tempObj[item.appId] && item.versionDesc) {
                item.versionDesc['targetVersion'] = tempObj[item.appId];
              }
              return item;
            })
            this.versionData = tempArr2;
          } else {
            this.$message.error(response.errorMsg);
          }
        })
        .catch(err => {});
    }
  }
};
</script>

<style scoped lang="less">
.split-info-box {
  margin-bottom: 20px;
  border-bottom: 1px solid #eee;
  font-size: 14px;
  .flex-box {
    display: flex;
    flex-wrap: wrap;
  }

  h3 {
    font-weight: normal;
    margin-bottom: 20px;
  }

  p {
    display: inline-block;
    width: 284px;
    margin-right: 20px;
    margin-bottom: 20px;
    line-height: 24px;

    span:nth-child(1) {
      font-size: 14px;
      color: #909399;
    }

    span:nth-child(2) {
      word-break: break-all;
      color: #333;
    }

    span.view-version-link {
      color: #409eff;
      cursor: pointer;
    }
  }
}
.software-version {
  max-width: 1000px;
  .version-title {
    margin-bottom: 10px;
    font-size: 14px;
    color: #909399;
  }
}
.no-border {
  border: none;
  margin-bottom: 0;
}
</style>
