<!-- eslint-disable vue/require-v-for-key -->
<template>
  <div class="home">
    <div class="wrap" v-show="loading">
      <swiper ref="mySwiper" :options="swiperOptions">
        <swiper-slide :class="{ 'swiper-no-swiping': !isSensor && !existAirSwitch }">
          <div class="page1">
            <ul class="jie-top clearfix">
              <li class="jie-left">
                <div class="jie-scenes">
                  <p>
                    <span class="scenes-icon"></span>
                    <span class="scenes-tit">场景</span>
                  </p>
                  <ul
                    v-if="scenes.length"
                    @touchmove="scenes && scenes.length > 6 ? $event.stopPropagation() : ''"
                    class="scenes-list scroll-wrap"
                    :class="{ 'limit-list': scenes && scenes.length > 6 }"
                  >
                    <li v-for="(item, index) in scenes" :key="index" @click="changeMode(item.id, scenes, index)" :class="{ active: item.active }">
                      <span>{{ item.name }}</span>
                    </li>
                  </ul>
                  <div class="data-null scenes-list" v-else>
                    <div>暂无场景数据</div>
                  </div>
                </div>
                <div class="jie-light jie-scenes">
                  <p>
                    <span class="scenes-icon light-icon"></span>
                    <span class="scenes-tit">空开</span>
                  </p>
                  <ul class="light-list">
                    <li v-show="false" class="light-switch" :class="{ close: !lightsAll }" @click="powerSwitch('light', !lightsAll)">
                      <p class="switch-icon"></p>
                      <p class="switch-tit">全开/关</p>
                    </li>
                    <li
                      class="lights"
                      :class="{
                        'light-switch light-bg': !item.enableBR,
                        mr45: brlength === 3,
                        mr30: brlength === 2,
                        mr15: brlength === 0,
                      }"
                      v-for="(item, index) in light"
                      :key="index"
                      @touchmove.stop
                    >
                      <div v-if="item.enableBR" @touchmove.stop>
                        <p>{{ item.name }}</p>
                        <input
                          orient="vertical"
                          type="range"
                          name="points"
                          min="0"
                          max="100"
                          step="1"
                          class="light-range"
                          @input="changeLight($event, index, item)"
                          :value="item.state ? item.brightness : 0"
                          @change="changeBR(item)"
                        />
                        <div class="light-hight" :style="`height: ${item.state ? item.brightness * height_flag : 0}%`"></div>
                      </div>
                      <div
                        class="light-hb light-switch"
                        :class="{
                          'hb-close close': !item.state,
                          mr15: brlength === 0,
                          ptlight: index,
                          'ptlight-close': index && !item.state,
                        }"
                        @click="singlePower(item, item.state)"
                        @touchmove.stop
                        v-else
                      >
                        <p class="switch-icon"></p>
                        <p class="switch-tit">{{ item.name }}</p>
                      </div>
                    </li>
                    <li class="line-slider" v-if="enableCCT || enableBR"></li>
                    <li class="lights" v-if="enableCCT" @touchmove.stop>
                      <p>色温</p>
                      <input
                        orient="vertical"
                        type="range"
                        name="points"
                        min="2700"
                        max="6500"
                        step="100"
                        class="light-range"
                        @input="changeLightCct($event)"
                        :value="lightsClose ? 0 : cct"
                        @change="changeCCT()"
                      />
                      <div class="light-hight light-cct" :style="`height: ${(lightsClose ? 0 : cct / 2700 - 1) * cct_flag}%`"></div>
                    </li>
                    <li class="lights" v-if="!enableCCT && enableBR">
                      <p>亮度</p>
                      <input
                        orient="vertical"
                        type="range"
                        name="points"
                        min="0"
                        max="100"
                        step="1"
                        class="light-range"
                        @input="changeLightBr($event)"
                        :value="lightsClose ? 0 : brightness"
                        @change="changeBr()"
                      />
                      <div class="light-hight light-cct" :style="`height: ${lightsClose ? 0 : brightness * height_flag}%`"></div>
                    </li>
                  </ul>
                </div>
              </li>
              <li class="jie-air jie-tits">
                <p>
                  <span class="scenes-icon air-icon"></span>
                  <span class="scenes-tit">空调</span>
                </p>
                <div v-if="aircon.length" class="air-wrap" @touchmove="aircon.length > 1 ? $event.stopPropagation() : ''">
                  <!-- :class="{ airX: aircon.length <= 1 }" -->
                  <div class="scroll-wrap">
                    <div class="wrap-scroll clearfix">
                      <div v-if="!airconStyleMode">
                        <div class="air-content" v-for="(item, index) in aircon" :key="index" :class="{ close: !item.state.state }">
                          <div>
                            <span class="air-title">{{ item.name }}</span>
                            <div class="air-increase">
                              <div class="air-screen" v-if="!v">{{ item.state.tt }}℃</div>
                              <div class="air-screen air-special" v-else>
                                <div class="wrap-air">
                                  <div class="air_icon"></div>
                                  <i>温度</i>
                                </div>
                                <div class="temp">
                                  {{ item.state.tt || 17 }}
                                  <i>℃</i>
                                </div>
                              </div>
                              <div class="air-button">
                                <p @click="changeTemp(item, 1)">
                                  <span class="add-icon"></span>
                                </p>
                                <p @click="changeTemp(item, -1)">
                                  <span class="less-icon"></span>
                                </p>
                              </div>
                            </div>
                            <ul class="air-control">
                              <li
                                v-for="(items, indexs) in item.airList"
                                :key="indexs"
                                :class="{
                                  special: items.id == 5,
                                  close: !items.active,
                                }"
                                @click="items.id == 5 ? airSwitch(item) : changeAirMode(item, items.mode, aircon, index)"
                              >
                                <p class="airall-icon" :class="[items.active && item.state.state ? 'o' + items.icon : 'c' + items.icon]"></p>
                                <p class="tit">{{ items.name }}</p>
                              </li>
                            </ul>
                          </div>
                        </div>
                      </div>
                      <div class="air-content" v-else>
                        <div class="ts-mode-aircon" v-for="(item, index) in aircon" :key="index">
                          <div class="mode-aircon">{{ item.name }}</div>
                          <ul class="air-control">
                            <li
                              v-for="(items, indexs) in item.airList"
                              :key="indexs"
                              :class="{
                                special: items.id == 5,
                                close: !items.active,
                              }"
                              @click="items.id == 5 ? airSwitch(item) : changeAirMode(item, items.mode, aircon, index)"
                            >
                              <p class="airall-icon" :class="[items.active && item.state.state ? 'o' + items.icon : 'c' + items.icon]"></p>
                              <p class="tit">{{ items.name }}</p>
                            </li>
                          </ul>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="zw-airp" v-if="!aircon.length && v">
                  <p class="zw-air"></p>
                  <P class="zw-wz">暂无空调设备</P>
                </div>
              </li>
            </ul>
            <div class="jie-curtain jie-tits">
              <p>
                <span class="scenes-icon"></span>
                <span class="scenes-tit">窗帘</span>
              </p>
              <div class="curtain-list">
                <div class="all-curtain" v-if="curtain.length">
                  <div class="curtain-switch" :class="{ close: !curtainAll.all }" @click="powerSwitch('curtain', 'open')">
                    <p class="switch-icon"></p>
                    <p class="switch-tit">全开</p>
                  </div>
                  <div class="curtain-switch" :class="{ close: curtainAll.all }" @click="powerSwitch('curtain', 'close')">
                    <p class="switch-icon"></p>
                    <p class="switch-tit">全关</p>
                  </div>
                  <div class="curtain-switch" :class="{ close: !curtainAll.paused }" @click="powerSwitch('curtain', 'paused')">
                    <p class="switch-icon"></p>
                    <p class="switch-tit">全暂停</p>
                  </div>
                </div>
                <div class="curtain-wrap" v-if="curtain.length" @touchmove="curtain.length > 2 ? $event.stopPropagation() : ''">
                  <div class="scroll-wrap curtain-special">
                    <div class="wrap-scroll">
                      <ul class="curtain-content" v-for="(item, index) in curtain" :key="index">
                        <li>
                          <span>{{ item.name }}</span>
                          <div class="curtain-switch" :class="{ close: item.state.state != 1 }" @click="changeState('open', item)">
                            <p class="switch-icon switch-open"></p>
                            <p class="switch-tit">拉开</p>
                          </div>
                          <div class="curtain-switch" :class="{ close: item.state.state != 0 }" @click="changeState('paused', item)">
                            <p class="switch-icon paused"></p>
                            <p class="switch-tit">暂停</p>
                          </div>
                          <div class="curtain-switch" :class="{ close: item.state.state != 2 }" @click="changeState('close', item)">
                            <p class="switch-icon switch-close"></p>
                            <p class="switch-tit">关起</p>
                          </div>
                        </li>
                      </ul>
                    </div>
                  </div>
                </div>
                <div v-if="!curtain.length" class="zw-curtain">
                  <P class="zw-icon"></P>
                  <p class="zw-wz">暂无窗帘设备</p>
                </div>
              </div>
            </div>
            <div class="c-loading" v-show="loadings">
              <div class="jie-loading">
                <div class="loading-icon">
                  <div class="round round1"></div>
                  <div class="round round2"></div>
                  <div class="round round3"></div>
                  <div class="round round4"></div>
                </div>
              </div>
            </div>
          </div>
        </swiper-slide>
        <swiper-slide v-if="isSensor">
          <div class="page2">
            <div class="jie-scenes">
              <p>
                <span class="scenes-icon env-icon"></span>
                <span class="scenes-tit">环境指数</span>
              </p>
            </div>
            <div class="sy-content">
              <div class="sy-c-left">
                <div class="echarts-env" v-if="sensorData.length">
                  <gauge-chart class="frast-chart" :dataSet="sensorData" />
                </div>
                <div class="l-tit">
                  <span>{{ suggest && suggest[0] }}</span>
                  <span>{{ lastTime ? lastTime + "前更新" : "暂无环境信息" }}</span>
                </div>
                <ul class="lenged">
                  <li class="item" v-for="(item, index) in lenged" :key="index">
                    <span>{{ item.name }}</span>
                    <span class="line" :style="item.color"></span>
                    <span>{{ item.value }}</span>
                  </li>
                  <li class="add-increase">
                    <span></span>
                    <span class="line"></span>
                    <span>100</span>
                  </li>
                </ul>
              </div>
              <div class="lines-sy"></div>
              <ul class="sy-c-right" v-if="sensorList.length">
                <li class="items" v-for="(item, index) in sensorList" :key="index">
                  <gauge-chart :id="`index${index}`" :type="true" :dataSet="[item]" />
                </li>
              </ul>
            </div>
          </div>
        </swiper-slide>
        <swiper-slide v-if="existAirSwitch">
          <div class="page3">
            <div class="jie-curtain jie-tits">
              <p>
                <span class="scenes-icon"></span>
                <span class="scenes-tit">智慧空开</span>
              </p>
              <div class="curtain-list" style="height: ;">
                <div v-for="item in airSwitchs">
                  <div class="curtain-switch" :class="{ close: !item.state.state }" @click="addrAirSwitch(item)">
                    <p class="switch-tit" style="color: #FFFFFF; font-weight: bold; font-size: 0.17rem; padding-top: 0.15rem;">
                      {{ item.state.dl }}
                    </p>
                    <p style="text-align:center;color: #FFFFFF; margin-bottom: 0.1em; font-size: 0.10rem;">
                      {{ item.state.dlunit }}
                    </p>
                    <p style=" margin-top: 0.01rem" class="switch-icon"></p>
                    <p class="switch-tit">开/关</p>
                  </div>
                  <p class="switch-tit" style="margin: 0.09rem;text-align: center; color: #9FA8BF; font-weight: 500;">
                    {{ item.alias_name }}
                  </p>
                </div>
              </div>
            </div>
          </div>
        </swiper-slide>
        <!-- 光感设置 -->
        <swiper-slide v-if="getLightSense && $hub.isLocalPrd">
          <LightSense :dev="getLightSense" :dev2="getPeopleSense" />
        </swiper-slide>
        <!-- 人感设置 -->
        <swiper-slide v-if="getPeopleSense && $hub.isLocalPrd">
          <PeopleSense :dev="getPeopleSense" />
        </swiper-slide>
        <div class="swiper-pagination" slot="pagination" v-show="isSensor || existAirSwitch"></div>
      </swiper>
    </div>
    <div class="re-fixed" v-show="loading">
      <div class="refresh" @click="reloads">
        <div class="icon-refresh"></div>
      </div>
    </div>
    <!-- || sensorData.length <= 0 -->
    <div v-show="!loading">
      <div class="jie-loading">
        <div class="loading-icon">
          <div class="round round1"></div>
          <div class="round round2"></div>
          <div class="round round3"></div>
          <div class="round round4"></div>
        </div>
        <div class="loading-text">{{ text }}</div>
      </div>
    </div>
  </div>
</template>

<script>
"use strict";
import { Swiper, SwiperSlide } from "vue-awesome-swiper";
import "swiper/css/swiper.css";
import GaugeChart from "@/components/charts/GaugeChart";
import { cache, comm, dateFormat } from "../utils";
import sensorKPI from "../utils/sensorKPI";
import config from "../config";
import auth from "../api/auth";
import basic from "../api/zone";
import scene from "../api/scene";
import device from "../api/device";
import event from "../api/event";
import { mapState } from "vuex";
import { Message } from "element-ui";
import LightSense from "./components/LightSense.vue";
import PeopleSense from "./components/PeopleSense.vue";
import { SetCurTime } from "@/api/sets.js";
// css 动态加载
const params = comm.resolveQuery();
params.v ? require(`../skin/${params.v}/index.less`) : require(`../skin/index.less`);
// 柱子的高度
const height_flag = params.v ? 5 : 3.3;
const cct_flag = params.v ? 360 : 240;
const v = params.v;
export default {
  name: "index",
  inject: ["reload"],
  data() {
    return {
      light: [],
      lightTemp: [],
      timeouts: null,
      cct: 2700,
      brightness: 80,
      loading: false,
      text: "正在加载...",
      loadings: false,
      time: 1500,
      swiperOptions: {
        noSwiping: true,
        pagination: {
          el: ".swiper-pagination",
        },
        // Some Swiper option/callback...
      },
      lenged: [
        {
          name: "较差",
          color: "background:rgba(250,137,137,1);",
          value: 0,
        },
        {
          name: "差",
          color: "background:rgba(241,188,2,1);",
          value: 30,
        },
        {
          name: "良好",
          color: "background:rgba(178,204,73,1);",
          value: 60,
        },
        {
          name: "优",
          color: "background:rgba(86,204,96,1);",
          value: 80,
        },
      ],
      enableCCT: 0,
      enableBR: 0,
      sensorData: [],
      sensorList: [],
      height_flag,
      cct_flag,
      v,
      airconStyleMode: window.staticConfig.airconStyleMode,
      // localprdTime: null,
    };
  },
  async mounted() {
    if (!this.$hub.isLocalPrd) {
      const params = comm.resolveQuery();
      if (params.u && params.p) {
        // 隐形登录
        await this.login(params.u, params.p);
        this.loading = true;
        // 请求数据，如果不存在的话
        // if (!this.zone) {
        await basic.getBasicInfo();
        this.synLocalprdTime(); // 进入同步时间
        // 判断是否可以调光调色， 现在逻辑是n个灯只要有一个可调色温显示调色温，都不能调色温的查看是否可以调亮度，只要有一个可以调亮度就显示调亮度
        this.enableCCT = this.lights.some((item) => Number(item.enableCCT));
        this.enableBR = this.lights.some((item) => Number(item.enableBR));
        this.refresh();
        //}
      } else {
        this.text = "请尽快与管理员联系,您的账号有误!";
      }
    } else {
      cache.setCache("dtit_user", window.info["user"][0]);
      cache.setSession("dtit_user", window.info["user"][0]);
      this.$nextTick(async () => {
        this.loading = true;
        await basic.getBasicInfo();
        await this.deviceState();
        this.synLocalprdTime(); // 进入同步时间
        this.enableCCT = this.lights.some((item) => Number(item.enableCCT));
        this.enableBR = this.lights.some((item) => Number(item.enableBR));
        setTimeout(() => {
          this.refresh();
        }, 3000);
        this.polling();
      });
      // 同步本地时间
      // this.localprdTime = setInterval(this.synLocalprdTime, 30000);
    }
  },
  // destroyed() {
  //   clearInterval(this.localprdTime);
  // },
  computed: {
    ...mapState(["devices", "scenes", "zone", "lights", "extDevices", "sensorTime", "sensorNd", "airState"]),
    curtain() {
      return this.extDevices.filter((item) => item.devicetype.key == "curtain");
    },
    curtainAll() {
      return {
        all: this.extDevices.filter((e) => e.devicetype.key === "curtain").every((l) => l.state.state === 1),
        paused: this.extDevices.filter((e) => e.devicetype.key === "curtain").every((l) => l.state.state === 0),
      };
    },
    lightsAll() {
      this.lightData();
      return this.lights.every((l) => l.state == 1);
    },
    lightsClose() {
      return this.lights.every((l) => l.state == 0);
    },
    // 获取有光感传感器
    getLightSense() {
      // type:传感器,devicenodetype：2代表光感
      return this.extDevices.find((d) => d.devicetype.type === 12 && d.devicenodetype === 2);
      // 暂时使用超感
      // return this.extDevices.find((d) => d.devicetype.type === 12 && d.devicenodetype === 1);
    },
    // 获取有人感传感器
    getPeopleSense() {
      // type:传感器,devicenodetype：3代表人感
      return this.extDevices.find((d) => d.devicetype.type === 12 && d.devicenodetype === 3);
    },
    aircon() {
      return this.airData();
    },
    swiper() {
      return this.$refs.mySwiper.$swiper;
    },
    lastTime() {
      let time = 0;
      if (this.$store.state.sensorTime) {
        let current = Math.abs(new Date().getTime() - new Date(this.$store.state.sensorTime).getTime());
        let days = current / 1000 / 60 / 60 / 24;
        let daysRound = Math.floor(days);
        let hours = current / 1000 / 60 / 60 - 24 * daysRound;
        let hoursRound = Math.floor(hours);
        let minutes = current / 1000 / 60 - 24 * 60 * daysRound - 60 * hoursRound;
        let minutesRound = Math.floor(minutes);
        time = daysRound ? `${daysRound}天` : hoursRound ? `${hoursRound}小时` : minutesRound ? `${minutesRound}分钟` : "当";
      }
      return time;
    },
    suggest() {
      return sensorKPI.getSuggest(this.$store.state.sensor);
    },
    // 是否存在传感器
    isSensor() {
      return this.extDevices.some((item) => item.devicetypeId == 7) || this.sensorNd.length;
    },
    // 可调光的数量
    brlength() {
      return this.lights.filter((item) => item.enableBR).length;
    },
    // 是否存在空开
    existAirSwitch() {
      return this.extDevices.some((item) => item.devicetype.key == "airswitch");
    },
    airSwitchs() {
      return this.extDevices.filter((item) => item.devicetype.key == "airswitch");
    },
  },
  components: {
    Swiper,
    SwiperSlide,
    GaugeChart,
    LightSense,
    PeopleSense,
  },
  methods: {
    // 同步本地时间
    async synLocalprdTime() {
      let res = await SetCurTime(dateFormat(new Date().getTime(), "yyyy-MM-dd hh:mm:ss"));
      if (res.data != 0) {
        Message.error("同步时间失败!");
      }
    },
    // 登录处理
    async login(u, p) {
      try {
        const result = await auth.login(u, p);
        if (!result.data[0].no) {
          result.data[0].no = u;
        }
        // 保存登录用户信息
        cache.setSession("dtit_user", result.data[0]);
      } catch (e) {
        this.text = e.message;
        this.loading = false;
        Message.error(e.message);
      }
    },
    changeLight(e, index, item) {
      let value = e.target.value;
      item.brightness = value;
      if (!item.state) item.state = true;
      this.$set(this.light, index, item);
      if (this.lightTemp.length) {
        this.$set(this.lightTemp, index, item);
      }
    },
    changeLightCct(e) {
      let value = e.target.value;
      this.cct = value;
    },
    changeLightBr(e) {
      let value = e.target.value;
      this.brightness = value;
    },
    // 切换场景
    async changeMode(id, scenes, idx) {
      // 开始激活场景，激活完成后，标记场景状态
      try {
        // 处理选中
        scenes.forEach((item, index) => {
          if (index == idx) {
            item.active = true;
          }
        });
        // this.commdDelay();
        // 本地化处理场景命令
        if (this.$hub.isLocalPrd) {
          // 对继电器版本的本地做特殊处理
          let cmd = JSON.parse(scenes[idx].cmd);
          cmd.forEach((item) => {
            if (item.nd >= 8 && item.nd <= 24) {
              item.nd = 255;
            }
          });
          this.loadings = true;
          await device._sendCommand([{ args: cmd }]);
          this.loadings = false;
        } else {
          await scene.active(id);
        }
        // 处理状态变化
        this.$store.commit("ACTIVE_SCENE", id);
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
      scenes.forEach((item, index) => {
        if (index == idx) {
          item.active = false;
        }
      });
    },
    // 窗帘
    async changeState(state, devices) {
      try {
        this.commdDelay();
        await device.singleControl(devices, state);
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // 灯数据
    lightData() {
      this.light = JSON.parse(JSON.stringify(this.lights));
      // 新版对灯通过sid分组处理
      if (this.devices && this.devices.length && this.devices[0]["version"] >= 45) {
        let arrTemp = [];
        // let arrName = ["第一组灯", "第二组灯", "第三组灯", "第四组灯"];
        let index = -1;
        this.light.forEach((item) => {
          if (arrTemp.every((itm) => itm.sid != item.sid) || !arrTemp.length) {
            ++index;
            arrTemp.push(item);
            // item.name = arrName[index];
            item.name = item.alias_name;
            item.nd = 255;
            item.tp = [1];
            let obj = this.lightTemp.find((itm) => itm.sid === item.sid);
            if (obj) {
              item.state = obj["brightness"] < 5 ? 0 : obj["state"];
              item.brightness = obj["brightness"];
            }
          }
        });
        this.light = arrTemp;
        this.lightTemp = arrTemp;
        clearTimeout(this.timeouts);
        this.timeouts = setTimeout(() => {
          this.light.forEach((item) => {
            let obj = this.lights.filter((itm) => itm.sid === item.sid);
            item.state = obj.every((itm) => itm.state);

            item.brightness = obj[0] && obj[0]["brightness"];
          });
        }, 3000);
      }
      this.cct = this.lights[0] && this.lights[0].cct;
      this.brightness = this.lights[0] && this.lights[0].brightness;
    },
    // 改变灯的亮度
    async changeBR(devices) {
      try {
        this.commdDelay();
        await device.singleControl(devices, "br", devices.brightness);
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // 改变色温
    async changeCCT() {
      try {
        this.commdDelay();
        await device.batchControls(["light"], "cct", Number(this.cct));
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // 改变亮度
    async changeBr() {
      try {
        this.commdDelay();
        await device.batchControl_light(["light"], "br", Number(this.brightness));
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // 全部开启或者全部关闭
    async powerSwitch(type, state) {
      try {
        this.commdDelay();
        if (type === "light") {
          await device.batchControl(["light"], !state ? "close" : "open");
          this.light.forEach((item) => {
            item.state = state;
          });
          if (this.lightTemp.length) {
            this.lightTemp.forEach((item) => {
              item.state = state;
            });
          }
        } else if (type === "curtain") {
          await device.batchControl(["curtain"], state);
        }
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // 单独电灯的开关
    async singlePower(devices, state) {
      try {
        this.commdDelay();
        await device.singleControl(devices, state ? "close" : "open");
        console.log(devices);
        // 处理状态
        if (this.lightTemp.length) {
          this.lightTemp.forEach((item, index) => {
            if (item.sid === devices.sid) {
              item.state = !state;
              this.$set(this.lightTemp, index, item);
              // 找出原始数据
              let lights = this.lights.filter((itm) => itm.sid === item.sid);
              lights.forEach((itm) => {
                itm.state = !state;
                this.$store.commit("UPDATE_LIGHT", {
                  nd: item.nd,
                  data: itm,
                });
              });
            }
          });
        }
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // 空调
    changeTemp(devices, value) {
      devices.timeout && clearTimeout(devices.timeout);
      devices.changing = true;
      devices.state.tt += value;
      if (devices.state.tt < 16) {
        devices.state.tt = 16;
      } else if (devices.state.tt > 30) {
        devices.state.tt = 30;
      }
      devices.timeout = setTimeout(() => {
        this.sendCommand(devices);
      }, 400);
    },
    async sendCommand(devices) {
      // 执行
      try {
        this.commdDelay();
        await device.singleControl(devices, "tt", devices.state.tt);
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      } finally {
        devices.changing = false;
      }
    },
    async changeAirMode(devices, mode) {
      try {
        // 预先处理状态
        let preCopy = JSON.parse(JSON.stringify(devices));
        let cg = {
          cool: 0,
          warm: 1,
          low: 1,
          medium: 2,
          high: 3,
        };
        if (["cool", "warm"].indexOf(mode) > -1) {
          devices.state.mode = cg[mode];
        } else {
          devices.state.air = cg[mode];
        }
        devices.state.state = 1;
        this.aircon.forEach((item, index) => {
          if (item.nd === devices.nd) {
            // console.log(this.aircon, index, devices);
            this.airButton([devices]);
            // console.log(devices);
            this.$set(this.aircon, index, devices);
            // console.log(this.aircon);
          }
        });
        this.commdDelay();
        await device.singleControl(preCopy, mode);
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    async airSwitch(devices) {
      try {
        // 预处理开关
        let preCopy = JSON.parse(JSON.stringify(devices));
        devices.state.state = !devices.state.state;
        this.aircon.forEach((item, index) => {
          if (item.nd === devices.nd) {
            this.$set(this.aircon, index, devices);
          }
        });
        this.commdDelay();
        await device.singleControl(preCopy, preCopy.state.state ? "close" : "open");
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // addr 空开
    async addrAirSwitch(devices) {
      try {
        // 预处理开关
        let preCopy = JSON.parse(JSON.stringify(devices));
        devices.state.state = !devices.state.state;
        this.commdDelay();
        await device.singleControl(preCopy, preCopy.state.state ? "close" : "open");
      } catch (e) {
        console.error(e.message);
        Message.error(e.message);
      }
    },
    // 空调数据
    airData() {
      const airConfig = [
        {
          name: "制冷",
          icon: "_cryogen",
          active: false,
          mode: "cool",
          id: 0,
        },
        {
          name: "制热",
          icon: "_heating",
          active: false,
          mode: "warm",
          id: 1,
        },
        {
          name: "低风",
          icon: "_low_wind",
          active: false,
          mode: "low",
          id: 1,
        },
        {
          name: "中风",
          icon: "_medium_wind",
          active: false,
          mode: "medium",
          id: 2,
        },
        {
          id: 5,
          name: "开/关",
          icon: "_switch",
          active: false,
          mode: "",
        },
        {
          name: "高风",
          icon: "_high_wind",
          active: false,
          mode: "high",
          id: 3,
        },
      ];
      const airConfig1 = [
        {
          id: 5,
          name: "开/关",
          icon: "_switch",
          active: false,
          mode: "",
        },
      ];
      let aircons = this.extDevices.filter((item) => item.devicetype.key == "aircon");
      aircons = JSON.parse(JSON.stringify(aircons));
      aircons.forEach((item) => {
        item.state.state = Number(item.state.state);
      });
      aircons.forEach((item) => {
        if (this.airconStyleMode) {
          item.airList = JSON.parse(JSON.stringify(airConfig1));
        } else {
          item.airList = JSON.parse(JSON.stringify(airConfig));
        }
      });
      this.airButton(aircons);
      return aircons;
    },
    // 空调按钮互斥
    airButton(air) {
      air.forEach((item) => {
        let { air, mode, state } = item.state;
        item.airList.forEach((items, idx) => {
          if (idx <= 1) {
            items.active = items.id === mode;
          } else if (items.id != 5) {
            items.active = items.id === air;
          } else {
            items.active = Number(state);
          }
        });
      });
    },
    // 延迟执行命令
    commdDelay() {
      this.loadings = true;
      let timer = null;
      clearTimeout(timer);
      timer = setTimeout(() => {
        this.loadings = false;
      }, this.time);
    },
    // 刷新
    reloads() {
      this.reload(); // 这种体验比location好
    },
    kpi() {
      let kpi = this.$store.state.sensor && parseInt(sensorKPI.getKPI(JSON.parse(JSON.stringify(this.$store.state.sensor))));
      let obj = {};
      if (kpi >= 80) {
        obj = { text: "优", color: "#56CC60" };
      } else if (kpi >= 60) {
        obj = { text: "良", color: "#B2CC49" };
      } else if (kpi >= 30) {
        obj = { text: "差", color: "#F1BC02" };
      } else {
        obj = { text: "较差", color: "#FA8989" };
      }
      return JSON.parse(
        JSON.stringify({
          kpi,
          text: obj.text,
          color: obj.color,
        })
      );
    },
    score() {
      return JSON.parse(
        JSON.stringify([
          {
            name: "E-IEQ指数",
            value: this.kpi().kpi,
            unit: "分",
            text: this.kpi().text,
            color: this.kpi().color,
            max: 100,
          },
        ])
      );
    },
    sensor() {
      let list = JSON.parse(JSON.stringify(this.$store.getters.sensorState(comm.sensorQuota())));
      let config = [
        { name: "温度", key: "DHTt", max: 100 },
        { name: "湿度", key: "DHTh", max: 100 },
        { name: "亮度", key: "ALS", max: 100 },
        { name: "PM2.5", key: "PM25", max: 5000 },
        { name: "PM10", key: "PM10", max: 5000 },
        { name: "CO2", key: "CO2", max: 10000 },
        { name: "甲醛", key: "CH2O", max: 5000 },
        { name: "TVOC", key: "TVOC", max: 10000 },
      ];
      list = config.map((item) => {
        return {
          name: item.name,
          value: list[item.key]["value"],
          unit: list[item.key]["unit"],
          text: list[item.key]["text"],
          color: list[item.key]["color"],
          max: item.max,
        };
      });
      return JSON.parse(JSON.stringify(list));
    },
    // 刷新数据
    refresh() {
      if (!this.isSensor) return;
      let time = null;
      clearInterval(time);
      this.sensorData = JSON.parse(JSON.stringify(this.score()));
      this.sensorList = JSON.parse(JSON.stringify(this.sensor()));
      time = setInterval(() => {
        this.sensorData = JSON.parse(JSON.stringify(this.score()));
        this.sensorList = JSON.parse(JSON.stringify(this.sensor()));
      }, 1000 * 60 * 10);
    },
    // 本地化获取设备的状态
    async deviceState() {
      // 灯
      if (window.staticConfig.isRelay) {
        // 如果是继电器版本灯的状态获取
        let result = await device.getDeviceState(255, "light");
        if (typeof result === "number") {
          result = String(result);
        }
        if (result.match(/^\d+$/g) && result.match(/^\d+$/g)[0]) return;
        result = result.replace(/'/g, '"');
        result = JSON.parse(result);
        let list = result.relay
          .toString(2)
          .split("")
          .reverse()
          .splice(0, 4);
        this.lights.forEach((item, index) => {
          event.updateDevice({
            data: { State: Number(list[index]), nd: item.nd, type: "light" },
            coreid: item.deviceId,
          });
        });
      } else {
        this.lights.forEach(async (item) => {
          let result = await device.getDeviceState(item.nd);
          if (typeof result === "number") {
            result = String(result);
          }
          if (result.match(/^\d+$/g) && result.match(/^\d+$/g)[0]) return;
          result = result.replace(/'/g, '"');
          result = JSON.parse(result);
          event.updateDevice({
            data: { ...result, type: "light" },
            coreid: item.deviceId,
          });
        });
      }

      // 外部设备
      this.extDevices.forEach(async (item) => {
        // 排除 窗帘、 面板 、 人感 、 光感
        if (
          ["curtain", "switch", "remote"].includes(item.devicetype.key) ||
          (item.devicetypeId === 19 && item.devicenodetype === 2) ||
          (item.devicetypeId === 19 && item.devicenodetype === 3)
        ) {
          return;
        }
        let result = await device.getDeviceState(item.nd);
        if (typeof result === "number") {
          result = String(result);
        }
        if (result.match(/^\d+$/g) && result.match(/^\d+$/g)[0]) return;
        result = result.replace(/'/g, '"');
        result = JSON.parse(result);
        if (result.tp === 11) {
          if (this.airState.length) {
            this.airState.forEach((item) => {
              if (item.nd === result.nd) {
                item = {
                  nd: result.nd,
                  State: result.State != undefined ? result.State : item.State,
                  mod: result.mod != undefined ? result.mod : item.mod,
                  temp: result.temp != undefined ? result.temp : item.temp,
                  spd: result.spd != undefined ? result.spd : item.spd,
                };
              }
            });
          }
        }
        if (!this.sensorNd[0] || !this.sensorNd[0]["nds"].includes(item.nd)) {
          result = this.transformDate(item.devicetype.key, result);
          event.updateDevice({
            data: { ...result, type: item.devicetype.key },
            coreid: item.deviceId,
          });
        } else {
          event.updateSensor({
            data: JSON.stringify({ ...result, type: item.devicetype.key }),
            coreid: item.deviceId,
            published_at: new Date(),
          });
        }
      });
    },
    // 转化格式
    transformDate(key, data) {
      switch (key) {
        case "aircon":
          return {
            nd: data.nd,
            nt: data.nt,
            tp: data.tp,
            id: data.id,
            state: data.State || 0,
            mode: data.mod === 1 ? 0 : 1,
            air: data.spd || 1,
            tt: data.temp || 16,
          };
        default:
          return data;
      }
    },
    // 轮训获取状态的值
    polling() {
      this.times = null;
      clearTimeout(this.times);
      this.times = setTimeout(() => {
        this.deviceState();
        this.polling();
      }, 1000 * config.staticConfig.time);
    },
  },
};
</script>
<style>
.frast-chart .data-value .value {
  font-size: 36px;
}
.data-null {
  height: 82px;
}
.data-null > div {
  width: 100%;
  text-align: center;
  line-height: 82px;
}
.zw-curtain {
  height: 96px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
.zw-wz {
  margin: 0px !important;
}
</style>
