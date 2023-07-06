<template>
  <div>
      <div ref="node2" style="height:160px;padding-top:6px;padding-left: 30px;padding-right:30px;border-bottom: #efefef solid 1px;color: #555555;font-size: 12px;">
          <div style="">
              <div style="line-height: 20px;">节点筛选：</div>
              <div>
                  <el-date-picker v-model="checked_year_date" type="daterange"     range-separator="至"
                                  start-placeholder="开始日期" end-placeholder="结束日期" @change="doFilter"></el-date-picker>
              </div>
              <div>
              <el-radio-group v-model="checked_sex" size="mini" style="margin-top:10px;" @change="doFilter">
                      <el-radio-button label="">全部</el-radio-button>
                      <el-radio-button label="2021"></el-radio-button>
                      <el-radio-button label="2022"></el-radio-button>
                      <el-radio-button label="2023"></el-radio-button>
                  </el-radio-group>

<!--              <el-slider v-model="checked_year_num" :min="2020" :max="2025" :step="1" @change="doFilter"></el-slider>-->
              <el-radio-group v-model="checked_isgoodman" size="mini" style="margin-left:50px;" @change="doFilter">
                  <el-radio-button label="">全部</el-radio-button>
                  <el-radio-button :label="true">个人作品</el-radio-button>
                  <el-radio-button :label="false">其他作品</el-radio-button>
              </el-radio-group>
          </div>
          <div class="filter2">
              <div style="line-height: 20px;">关系筛选：</div>
              <el-checkbox-group v-model="state.rel_checkList" @change="doFilter">
                  <el-checkbox v-for="thisItem in all_rel_type" :key="thisItem" :label="thisItem" />
              </el-checkbox-group>
          </div>
      </div>
      </div>
      <div ref="myPageRef" style="border: #efefef solid 1px; height: calc(100vh - 100px);width: 100%;">
        <relation-graph ref="relationGraph$" :options="options">
          <template #node="{node}">
            <div style="padding-top:20px;" @mouseover="showNodeTips(node, $event)" @mouseout="hideNodeTips(node, $event)">{{node.text}}</div>
          </template>
        </relation-graph>
      </div>
      <div v-if="isShowNodeTipsPanel" :style="{left: nodeMenuPanelPosition.x + 'px', top: nodeMenuPanelPosition.y + 'px' }" style="z-index: 999;padding:10px;background-color: #ffffff;border:#eeeeee solid 1px;box-shadow: 0px 0px 8px #cccccc;position: absolute;">
          <div style="line-height: 25px;padding-left: 10px;color: #888888;font-size: 12px;">歌曲名称：{{currentNode.text}}</div>
          <div class="c-node-menu-item">Type: {{currentNode.data.type}}</div>
          <div class="c-node-menu-item">Date: {{currentNode.data.date}}</div>
      </div>
  </div>
</template>



<script setup lang="ts">
import { onMounted, ref,reactive,Ref,defineComponent } from 'vue'
import RelationGraph, {RGJsonData} from 'relation-graph/vue3'
import watermark from './WaterMark'
const checked_sex = ref('');
const checked_isgoodman = ref('');
const checked_year_date = ref([]);
const isShowCodePanel = ref(false);
const isShowNodeTipsPanel = ref(false);
const nodeMenuPanelPosition = ref({ x: 0, y: 0 });
const currentNode = ref({});
const melody_color = ref('rgba(0, 206, 209, 1)');
console.log(melody_color.value)
const  state = reactive({
    rel_checkList: ['作词', '作曲', '作词*', '7Senses', '拜托你了朋友', '生日公演']
});
const all_rel_type = ['作词', '作曲', '作词*', '7Senses', '拜托你了朋友', '生日公演']
const myPageRef: Ref<HTMLElement | null> = ref(null);
const relationGraph$ = ref<RelationGraph>()

const options = {
  defaultExpandHolderPosition: 'right',
  // defaultLineShape: 4,
  debug: true,
  showDebugPanel: true,
}
const  convertToDate = (dateString) => {
    if (!dateString) {
        return null; // 返回 null 或其他适当的空值表示日期无效
    }
    // 自定义函数将字符串日期转换为实际日期对象
    const parts = dateString.split('-');
    const year = parseInt(parts[0], 10);
    const month = parseInt(parts[1], 10) - 1; // 月份从 0 开始
    const day = parseInt(parts[2], 10);
    if (isNaN(year) || isNaN(month) || isNaN(day)) {
        return null; // 返回 null 或其他适当的空值表示日期无效
    }

    return new Date(year, month, day);
}
const showNodeTips = (nodeObject, $event) =>{
    currentNode.value = nodeObject;
    const _base_position = myPageRef.value?.getBoundingClientRect();
    console.log('showNodeMenus:', $event, _base_position);
    isShowNodeTipsPanel.value = true;
    nodeMenuPanelPosition.value.x = $event.clientX - _base_position.x + 10;
    nodeMenuPanelPosition.value.y = $event.clientY - _base_position.y + 10;

    console.log('x-y', $event.clientX)
}
const hideNodeTips = (nodeObject, $event) => {
    isShowNodeTipsPanel.value = false;
}


const doFilter = () => {
    const allNodes = relationGraph$.value.getInstance().getNodes();
    const allLinks = relationGraph$.value.getInstance().getLinks();
    allNodes.forEach(node => {
        let isHideThisLine = false;
        if (checked_sex.value !== '') {
            if (node.data['year'] !== checked_sex.value) {
                isHideThisLine = true;
            }
        }
        if (checked_isgoodman.value !== '') {
            if (node.data['isOriginal'] !== checked_isgoodman.value) {
                isHideThisLine = true;
            }
        }
        if (checked_year_date.value !== []) {
                const [start_time,end_time] = checked_year_date.value;
            const time = convertToDate(node.data['date']);
            if (time < start_time || time >end_time)  {
                isHideThisLine = true;
            }
        }
        node.opacity = isHideThisLine ? 0.1 : 1;
    });
    allLinks.forEach(thisLink => {
        thisLink.relations.forEach(thisLine => {
            // console.log(thisLine.data['type'])
            // console.log(state.rel_checkList.indexOf(thisLine.data['type'])=== -1)
            if (state.rel_checkList.indexOf(thisLine.data['type']) === -1) {
                if (!thisLine.isHide) {
                    thisLine.isHide = true;
                    console.log('Hide line:', thisLine);
                }
            } else {
                if (thisLine.isHide) {
                    thisLine.isHide = false;
                    console.log('Show line:', thisLine);
                }
            }
        });
        // node.opacity = isShowThisNode ? 1 : 0.1
    });
    relationGraph$.value.getInstance().dataUpdated();
};
onMounted(() => {
    watermark.set('图谱展示demo','许杨玉琢作品汇总')
  const graphJsonData:RGJsonData = {
    rootId: 'N3',
    nodes: [
        { id: 'N3', text: '许杨玉琢',borderColor: 'yellow'  },
        { id: 'N4', text: 'If U Never Gonna Try','data':{'type':'个人作品','isOriginal':true,'year':"2021",'date':'2021-05-01'},borderWidth: -1},
        { id: 'N5', text: '郁合','data':{'type':'个人作品','isOriginal':true,'year':"2021",'date':'2021-06-20'} ,borderWidth: -1},
        { id: 'N6', text: 'Rocket','data':{'type':'合作作品','isOriginal':false,'year':"2023",'date':'2023-05-27'} ,borderWidth: -1},
        { id: 'N7', text: 'Encore','data':{'type':'7Senses','isOriginal':false,'year':"2022",'date':'2022-11-21'} ,borderWidth: -1},
        { id: 'N8', text: 'Our Party','data':{'type':'个人作品','isOriginal':true,'year':"2022",'date':'2022-12-11'},borderWidth: -1 },
      { id: 'N9', text: 'All About U','data':{'type':'个人作品','isOriginal':true,'year':"2023",'date':'2023-04-06'} ,borderWidth: -1},
      { id: 'N10', text: 'Tell Me Why','data':{'type':'个人作品','isOriginal':true,'year':"2023",'date':'2023-06-17'} ,borderWidth: -1},
      { id: 'N11', text: '曼陀罗/Hate 2 Love','data':{'type':'音乐制作参与','isOriginal':false,'year':"2022",'date':'2022-06-13'} ,borderWidth: -1},
      { id: 'N12', text: 'Eliwa Trend','data':{'type':'生日公演','isOriginal':true,'year':"2023",'date':'2023-06-17'} ,borderColor: 'yellow' },
    ],
    lines: [
        { from: 'N3', to: 'N4', text: '作词','data': { 'type': '作词' },'color': 'rgba(255, 120, 0, 1)',
            'lineWidth': 1},
        { from: 'N3', to: 'N4', text: '作曲','data': { 'type': '作曲' } ,'color': melody_color.value,
            'lineWidth': 1},
        { from: 'N3', to: 'N9', text: '作词','data': { 'type': '作词' },'color': 'rgba(255, 120, 0, 1)',
            'lineWidth': 1 },
        { from: 'N3', to: 'N9', text: '作曲' ,'data': { 'type': '作曲' },'color': melody_color.value,
            'lineWidth': 1},
        { from: 'N3', to: 'N10', text: '作词','data': { 'type': '作词' },'color': 'rgba(255, 120, 0, 1)',
            'lineWidth': 1 },
        { from: 'N3', to: 'N10', text: '作曲' ,'data': { 'type': '作曲' },'color': melody_color.value,
            'lineWidth': 1},
        { from: 'N3', to: 'N11', text: '作词','data': { 'type': '作词' },'color': 'rgba(255, 120, 0, 1)',
            'lineWidth': 1 },
        { from: 'N3', to: 'N11', text: '有🐑唱的音源' ,'data': { 'type': '有🐑唱的音源' },'color': melody_color.value,
            'lineWidth': 1},
      { from: 'N3', to: 'N5', text: '作词','data': { 'type': '作词' },'color': 'rgba(255, 120, 0, 1)',
          'lineWidth': 1 },
      { from: 'N3', to: 'N5', text: '作曲' ,'data': { 'type': '作曲' },'color': melody_color.value,
          'lineWidth': 1},
      { from: 'N3', to: 'N6', text: '作曲','data': { 'type': '作曲' } ,'color': melody_color.value,
          'lineWidth': 1},
      { from: 'N3', to: 'N6', text: '作词*','data': { 'type': '作词*' },'color': 'rgba(255, 120, 0, 1)',
          'lineWidth': 1 },
      { from: 'N3', to: 'N7', text: '7Senses','data': { 'type': '7Senses' } },
      { from: 'N9', to: 'N8', text: '拜托你了朋友' ,'data': { 'type': '拜托你了朋友' }},
      { from: 'N10', to: 'N12', text: '生日公演' ,'data': { 'type': '生日公演' },'color':melody_color.value}
    ],
  };
  relationGraph$.value.setJsonData(graphJsonData, () => {
    console.log('relationGraph ready!');
  })
})


</script>

<style>

</style>

<style scoped>
.c-node-menu-item{
  line-height: 30px;padding-left: 10px;cursor: pointer;color: #444444;font-size: 14px;border-top:#efefef solid 1px;
}
.c-node-menu-item:hover{
  background-color: rgba(66,187,66,0.2);
}
.filter2{
    padding-bottom: 30px; /* 顶部间距大小，根据需要调整 */
}
</style>

