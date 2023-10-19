<template>
  <div v-if="extra.bol" class="post_list">
    <div v-if="cData.length != 0">
      <div class="box" v-for="(item, index) in cData" :key="item.id.toString()">
        <div class="header h" v-if="!extra.showBtn">
          No.{{ index + 1 }}|id:{{ item.id.toString() }}|create_time:{{
            item.create_time
          }}
        </div>
        <div class="header h" v-if="extra.showBtn">
          No.{{ index + 1 }}|id:{{ item.id.toString() }}|create_time:{{
            formatTime(item.create_time)
          }}
        </div>
        <div class="body" :key="dayNum">
          <video
            controls
            muted
            :src="item.media[0].url"
            v-if="!extra.showBtn"
          ></video>
          <img :src="item.cover_img" alt="图片" v-if="extra.showBtn" />
          <div class="field" v-if="!extra.showBtn">
            <div class="h">delete_time:{{ item.delete_time }}</div>
            <div class="h">spam_flag:{{ item.spam_flag }}</div>
            <div class="h">
              spam_recommend_flag:{{ item.spam_recommend_flag }}
            </div>
            <div class="h">
              feed_analyze_block_level:{{ item.feed_analyze_block_level }}
            </div>
            <div class="h">flag:{{ item.flag }}</div>
            <div class="h">like_count:{{ item.like_count }}</div>
            <div class="h">comment_count:{{ item.comment_count }}</div>
          </div>
          <div class="field" v-if="extra.showBtn">
            <div class="h">live_id:{{ item.live_id.toString() }}</div>
            <div class="h">
              总观看人数:{{ item.live_stats.total_audience_count }}
            </div>
            <div class="h">喝彩数:{{ item.live_stats.total_cheer_count }}</div>
            <div class="h">
              直播时长:{{ item.live_stats.live_duration_in_seconds }}
            </div>
            <div class="h">
              评论数:{{ item.live_stats.total_comment_count }}
            </div>
            <div class="h">
              最高在线:{{ item.live_stats.total_audience_count }}
            </div>
            <div class="h">推拉流状态:{{ item.with_source_stream }}</div>
            <div class="h">禁言状态:{{ item.room_muted }}</div>
            <div class="h">打赏状态:{{ item.reward_enabled }}</div>
            <div class="h">连麦状态:{{ item.mic_enable }}</div>
            <div class="h">付费人数:undefined</div>
          </div>
        </div>
        <div v-if="extra.showBtn">
          <div class="options_button" v-if="index == 0">
            <span title="禁言本场直播" class="btn" @click="clickBtn(0, index)"
              >禁言本场直播</span
            >
            <span title="取消禁言" class="btn" @click="clickBtn(1, index)"
              >取消禁言</span
            >
            <span
              title="关闭本场直播"
              class="btn no"
              @click="clickBtn(2, index)"
              >关闭本场直播</span
            >
            <span
              title="强制关闭直播"
              class="btn no"
              @click="clickBtn(3, index)"
              >强制关闭直播</span
            >
          </div>
          <div class="options_button">
            <span
              title="查看本场直播曲线"
              class="btn"
              @click="clickBtn(4, index)"
              >查看本场直播曲线</span
            >
            <span title="混流" class="btn" @click="clickBtn(5, index)"
              >混流</span
            >
            <span title="直播" class="btn" @click="clickBtn(6, index)"
              >直播</span
            >
            <span title="直播回放地址" class="btn" @click="clickBtn(7, index)"
              >直播回放地址</span
            >
          </div>
        </div>
        <div class="description">
          {{ item.description }}
        </div>
        <div v-if="extra.topic && item.hash_tag_info">
          <div>话题词展示：</div>
          <div
            class="more"
            v-for="(topicItem, topicIndex) in item.hash_tag_info.topic_info"
            :key="topicIndex"
            @click="clickTopic(index, topicIndex)"
          >
            <span>topic:{{ topicItem.topic }}</span
            ><span>》</span>
          </div>
        </div>
        <div class="more" @click="clickMore(index)">
          <span>更多详情</span><span>》</span>
        </div>
        <div class="address" v-if="item.online_replay_url">
          直播回放地址：{{ item.online_replay_url }}
        </div>
      </div>
      <MyPagination
        :pageNo="this.pageNo"
        :pageSize="this.pageSize"
        :total="extra.object_id_list.length"
        :continues="5"
        @getPageNo="getPageNo"
      />
    </div>
    <div v-if="cData.length === 0">数据为空</div>
  </div>
</template>

<script>
import MyPagination from "./MyPagination.vue";
import JSONbig from "json-bigint";
const dayjs = require("dayjs");
export default {
  name: "PostList",
  props: ["extra"],
  inject: {
    _currentPageInstance: {
      default: {},
    },
  },
  components: {
    MyPagination,
  },
  data() {
    return {
      pageNo: 1,
      pageSize: 10,
      dayNum: dayjs().valueOf(),
      url: "",
      list: [],
    };
  },
  watch: {
    "extra.bol"() {
      this.pageNo = 1;
      this.pageSize = 10;
    },
  },
  computed: {
    cData: {
      get() {
        return this.list.length ? this.list : this.extra.data;
      },
    },
  },
  methods: {
    formatTime(time) {
      return dayjs.unix(time).format("YYYY-MM-DD HH:mm:ss");
    },
    getPageNo(pageNo) {
      clf.loading.show();
      this.pageNo = pageNo;
      if (this.extra.object_id_list.length > 10) {
        clf.api.gameauthadminweb
          .getFinderObjectByBroker({
            req_type: 0,
            url_scene: 1,
            uin: this.extra.detail.userattr.wx_uin,
            finder_uin: String(this.extra.detail.finder_uin),
            object_id_list: this.extra.object_id_list.slice(
              (this.pageNo - 1) * this.pageSize,
              this.pageSize * this.pageNo
            ),
          })
          .then((res) => {
            let value = JSONbig.parse(res.data.result_json).object.reverse();
            value = value.map((i) => {
              i.create_time = dayjs
                .unix(i.create_time)
                .format("YYYY-MM-DD HH:mm:ss");
              return i;
            });
            this.dayNum = dayjs().valueOf();
            this.list = value;
          })
          .finally(() => {
            clf.loading.close();
          });
      }
    },
    clickBtn(e, index) {
      let value = this.cData[index];
      let detail = this.extra.detail;
      switch (e) {
        case 0:
          clf.api.gameauthadminweb
            .setLiveMuteStatus({
              anchor_finder_uin: value.finder_uin,
              live_object_id: value.live_object_id,
              room_muted: 1,
            })
            .then((res) => {
              console.log(res, "res");
            });
          break;
        case 1:
          clf.api.gameauthadminweb
            .setLiveMuteStatus({
              anchor_finder_uin: value.finder_uin,
              live_object_id: value.live_object_id,
              room_muted: 0,
            })
            .then((res) => {
              console.log(res, "res");
            });
          break;
        case 2:
          break;
        case 3:
          console.log(3);
          break;
        case 4:
          window.open(
            `https://mmfinder.woa.com/wego/wevideoacctweb/page/getlivedata?object_id=${value.live_object_id}`
          );
          break;
        case 5:
          window.open(
            `https://mmfinder.woa.com/wego/wevideoacctweb/page/ForcePush?live_id=${value.live_object_id}&wx_uin=${detail.userattr.wx_uin}`
          );
          break;
        case 6:
          window.open(
            `https://mmfinder.woa.com/wego/wevideoacctweb/page/ForcePush?object_id=object_id=${value.live_object_id}&finder_uin=${detail.finder_uin}`
          );
          break;
        case 7:
          clf.api.gameauthadminweb
            .getFinderObject({
              feed_id: value.live_object_id,
              tcoa_ticket:
                "TOF4TeyJ2IjoiNCIsInRpZCI6Ikp2cndTeGNKQk1KbWNyaFB3elhVdWFwUmpEMk5MTGVuIiwiaXNzIjoiMTAuOTkuMjA4LjU3IiwiaWF0IjoiMjAyMi0xMC0xNFQxMDo0NTozMi45MzM1MjMwNTErMDg6MDAiLCJhdWQiOiIzMC4xNi40OS4zMCIsImhhc2giOiI0M0Y2Qzc5QjE2NzhERTlDM0M0RTU3QjczNTQ2QjJEMUFGMDExOEYwQ0QyNEMxRDAxNzQ3NjM3MTlFRTdDNzY4IiwibmgiOiJEMDNBNzA1OEI1ODgyOUEwMjJDNjM2RUU1RTgyRDc4OUEyNkI4QkY5NUY5NjNBMUQwRkMzODBEMjAzOUU2NEY2In0",
            })
            .then((res) => {
              let url = JSONbig.parse(res.data.result_json).object.live_info
                .online_replay_url;
              Vue.set(this.list[index], "online_replay_url", url);
            });
          break;
      }
    },
    clickMore(index) {
      let value = this.cData[index].id.toString();
      this.$router.push({
        path: "/mmfinder/feed/feedinfo",
        query: {
          feed_id: value,
        },
      });
    },
    //点击话题词
    clickTopic(index, topicIndex) {
      let value = this.cData[index].hash_tag_info.topic_info[topicIndex];
      window.open(
        `/mmfinder/#/mmfinder/feed/GetTopicInfo?topic=${
          value.topic
        }&topic_id=${value.topic_id.toString()}&topic_type=${value.topic_type}`
      );
    },
  },
};
</script>
<style lang="less" scoped>
@import "./styles/DraftBox.less";
</style>
