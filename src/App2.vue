<template>
    <div>
        <div style="height:110px;padding-top:6px;padding-left: 30px;padding-right:30px;border-bottom: #efefef solid 1px;color: #555555;font-size: 12px;">
            <el-button type="success" class="c-show-code-button"><el-link href="https://github.com/seeksdream/relation-graph/blob/master/examples/views/seeks-graph-docs/demo/Demo4AdvDataFilter.vue" target="_blank" style="color: #ffffff;">查看代码</el-link></el-button>
            <div style="">
                <div style="line-height: 20px;">节点筛选：</div>
                <el-radio-group v-model="checked_sex" size="mini" @change="doFilter">
                    <el-radio-button label="">全部</el-radio-button>
                    <el-radio-button label="男"></el-radio-button>
                    <el-radio-button label="女"></el-radio-button>
                </el-radio-group>
                &nbsp;&nbsp;&nbsp;&nbsp;
                <el-radio-group v-model="checked_isgoodman" size="mini" style="margin-left:50px;" @change="doFilter">
                    <el-radio-button label="">全部</el-radio-button>
                    <el-radio-button :label="true">正面人物</el-radio-button>
                    <el-radio-button :label="false">反面人物</el-radio-button>
                </el-radio-group>
            </div>
            <div>
                <div style="line-height: 20px;">关系筛选：</div>
                <el-checkbox-group v-model="rel_checkList" @change="doFilter">
                    <el-checkbox v-for="thisItem in all_rel_type" :key="thisItem" :label="thisItem" />
                </el-checkbox-group>
            </div>
        </div>
        <div style="margin-top:0px;width: calc(100% - 10px);height:calc(100vh - 200px);">
            <RelationGraph ref="seeksRelationGraph" :options="graphOptions" />
        </div>
    </div>
</template>

<script>
import { ref } from 'vue';

export default {
    name: 'SeeksRelationGraphDemo',
    components: { }, // 如果您没有在main.js文件中使用Vue.use(RelationGraph); 就需要在这里注册：components: { RelationGraph }
    setup() {
        const gLoading = ref(true);
        const demoName = ref('---');
        const checkedSex = ref('');
        const checkedIsGoodMan = ref('');
        const relCheckList = ref(['师生', '上下级', '亲戚', '情人', '朋友', '夫妻', '勾结', '腐化', '举报']);
        const allRelType = ref(['师生', '上下级', '亲戚', '情人', '朋友', '夫妻', '勾结', '腐化', '举报']);
        const graphOptions = ref({
            defaultNodeBorderWidth: 0,
            defaultNodeColor: 'rgba(238, 178, 94, 1)',
            allowSwitchLineShape: true,
            allowSwitchJunctionPoint: true,
            defaultLineShape: 1,
            layouts: [
                {
                    label: '自动布局',
                    layoutName: 'force',
                    layoutClassName: 'seeks-layout-force',
                },
            ],
            defaultJunctionPoint: 'border',
            // 这里可以参考"Graph 图谱"中的参数进行设置
        });

        const setGraphData = () => {
            const graphJsonData = {};
            console.log(this)
            const graphInstance = this.seeksRelationGraph.setJsonData(graphJsonData, () => {
                // 这些写上当图谱初始化完成后需要执行的代码
            });
        };

        const doFilter = () => {
            const allNodes = this.seeksRelationGraph.getInstance().getNodes();
            const allLinks = this.seeksRelationGraph.getInstance().getLinks();
            allNodes.forEach(node => {
                let isHideThisLine = false;
                if (checkedSex.value !== '') {
                    if (node.data['sexType'] !== checkedSex.value) {
                        isHideThisLine = true;
                    }
                }
                if (checkedIsGoodMan.value !== '') {
                    if (node.data['isGoodMan'] !== checkedIsGoodMan.value) {
                        isHideThisLine = true;
                    }
                }
                node.opacity = isHideThisLine ? 0.1 : 1;
            });
            allLinks.forEach(link => {
                link.relations.forEach(line => {
                    if (relCheckList.value.indexOf(line.data['type']) === -1) {
                        if (!line.isHide) {
                            line.isHide = true;
                            console.log('Hide line:', line);
                        }
                    } else {
                        if (line.isHide) {
                            line.isHide = false;
                            console.log('Show line:', line);
                        }
                    }
                });
                // node.opacity = isShowThisNode ? 1 : 0.1
            });
            this.seeksRelationGraph.getInstance().dataUpdated();
        };

        return {
            gLoading,
            demoName,
            checkedSex,
            checkedIsGoodMan,
            relCheckList,
            allRelType,
            graphOptions,
            setGraphData,
            doFilter,
        };
    },
    created() {},
    mounted() {
        console.log(this)
        // this.demoName = this.$route.params.demoname;
        this.setGraphData();
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.c-my-node{
    background-position: center center;
    background-size: 100%;
    border:#ff8c00 solid 2px;
    height:80px;
    width:80px;
    border-radius: 40px;
}
.c-node-name{
    width:160px;margin-left:-40px;text-align:center;margin-top:85px;
}
</style>