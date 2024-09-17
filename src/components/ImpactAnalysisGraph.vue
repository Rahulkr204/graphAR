<template>
    <div class="vue-flow-wrapper">
        <div class="mb-4 text-4xl font-medium">Impact Analysis</div>
        <div class="flex flex-row w-full">
            <input type="text" v-model="searchTerm" placeholder="Enter node number..."
                @change="e => searchTerm = e.target.value"
                class="px-2 py-1 mr-2 bg-white border border-blue-400 border-solid rounded outline-none basis-11/12" />
            <button @click="searchNode"
                class="text-white bg-blue-500 border border-blue-500 border-solid rounded basis-1/12">Search</button>
        </div>

        <VueFlow v-model="elements" :default-viewport="{ zoom: 0.5 }" :fit-view-on-init="true">
            <Background pattern-color="#e2e2e2" gap="8" />
            <Controls>
                <ControlButton @click="onFitView">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                        <path
                            d="M15 3l2.3 2.3-2.89 2.87 1.42 1.42L18.7 6.7 21 9V3zM3 9l2.3-2.3 2.87 2.89 1.42-1.42L6.7 5.3 9 3H3zm6 12l-2.3-2.3 2.89-2.87-1.42-1.42L5.3 17.3 3 15v6zm12-6l-2.3 2.3-2.87-2.89-1.42 1.42 2.89 2.87L15 21h6z" />
                    </svg>
                </ControlButton>
            </Controls>

            <template #node-custom="{ data }">
                <div :style="{ padding: '10px', borderRadius: '3px', border: '1px solid #ddd', background: data.level === 0 ? '#e1e1e1' : '#fff', color: '#000' }"
                    @mouseenter="showTooltip($event, data.description)" @mouseleave="hideTooltip">
                    {{ data.label }}
                </div>
            </template>
        </VueFlow>
    </div>
    <!-- <div v-show="tooltipVisible" class="custom-tooltip" :style="tooltipStyle">
        {{ tooltipContent }}
    </div> -->
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { VueFlow, useVueFlow } from '@vue-flow/core';
import { Background } from '@vue-flow/background';
import { ControlButton, Controls } from '@vue-flow/controls';
import { MiniMap } from '@vue-flow/minimap';

import graphData from './graphData.json';

import '@vue-flow/core/dist/style.css';
import '@vue-flow/core/dist/theme-default.css';
import '@vue-flow/controls/dist/style.css';
import '@vue-flow/minimap/dist/style.css';

const elements = ref([]);
const data = ref([]);
const searchTerm = ref('');
const { onPaneReady, fitView } = useVueFlow();

const tooltipVisible = ref(false);
const tooltipContent = ref('');
const tooltipStyle = ref({
    position: 'fixed',
    top: '0px',
    left: '0px'
});

const searchNode = async () => {
    const response = await fetch(`https://fea5-3-110-12-203.ngrok-free.app/api/v1/report/atlan/Table/impact_analysis_report?internalID=${searchTerm.value || 1}`, {
        method: 'POST', // Specify POST method
        headers: {
            'Content-Type': 'application/json', // Set the content type
            // Include other headers if needed
        },
        body: JSON.stringify({
            "change_type": "ADD_NEW_FIELD",
            "version": "1.0.1",
        })
    })
    const data = await response.json();
    elements.value = processData(data)
};

const showTooltip = (event, description) => {
    tooltipContent.value = description;
    tooltipStyle.value.top = `${event.clientY + 10}px`;
    tooltipStyle.value.left = `${event.clientX + 10}px`;
    tooltipVisible.value = true;
};

const hideTooltip = () => {
    tooltipVisible.value = false;
};

const impactColorMap = {
    0: '#000000',
    1: '#0000FF',
    2: '#FFA500',
    3: '#FF0000'
};

const radialLayout = (nodes, centerX, centerY, radius) => {
    const angleStep = (2 * Math.PI) / nodes.length;
    return nodes.map((node, index) => {
        const angle = index * angleStep;
        const x = centerX + radius * Math.cos(angle);
        const y = centerY + radius * Math.sin(angle);
        return { ...node, position: { x, y } };
    });
};

const nodeStyle = (data) => ({
    padding: '10px',
    borderRadius: '3px',
    border: '1px solid #ddd',
    background: data.level === 0 ? '#e1e1e1' : '#fff',
    color: '#000',
    width: '150px',
    textAlign: 'center'
});

const processData = (data) => {
    const nodes = [];
    const edges = [];
    const centerX = 500;
    const centerY = 300;
    const radius = 250;

    data.impactAnalyzer.forEach((analyzer, analyzerIndex) => {
        analyzer.connectedVertices.forEach(level => {
            level.levelConnectedVertices.forEach((vertex) => {
                const nodeId = `${analyzerIndex}-${vertex.id}`;
                nodes.push({
                    id: nodeId,
                    type: 'custom',
                    data: {
                        label: vertex.assetId,
                        level: vertex.level,
                        description: vertex.description || 'No description available'
                    }
                });

                vertex.connections.forEach(connection => {
                    const targetId = `${analyzerIndex}-${connection.id}`;
                    nodes.push({
                        id: targetId,
                        type: 'custom',
                        data: {
                            label: connection.assetId,
                            level: connection.level,
                            description: connection.description || 'No description available'
                        }
                    });

                    edges.push({
                        id: `e${nodeId}-${targetId}`,
                        source: nodeId,
                        target: targetId,
                        style: { stroke: impactColorMap[connection.impactColor] },
                        animated: connection.impactColor !== 0,
                        label: connection.description
                    });
                });
            });
        });
    });

    const layoutedNodes = radialLayout(nodes, centerX, centerY, radius);
    return [...layoutedNodes, ...edges];
};

const onFitView = () => {
    fitView();
};

onMounted(async () => {
    // Your existing data loading code here
    // Make sure to include a 'description' field in your connection objects
    // const rawData = {
    //     "impactAnalyzer": [
    //         {
    //             "connectedVertices": [
    //                 {
    //                     "level": 0,
    //                     "levelConnectedVertices": [
    //                         {
    //                             "assetId": "connector_last_checked",
    //                             "connections": [
    //                                 {
    //                                     "assetId": "_fivetran_job_last_checked",
    //                                     "edgeType": "ops",
    //                                     "graphName": "phoenix",
    //                                     "id": 12,
    //                                     "impactColor": 0,
    //                                     "level": 1,
    //                                     "description": "Description for _fivetran_job_last_checked"
    //                                 },
    //                                 {
    //                                     "assetId": "transaction_process_rank",
    //                                     "edgeType": "ops",
    //                                     "graphName": "phoenix",
    //                                     "id": 37,
    //                                     "impactColor": 0,
    //                                     "level": 1,
    //                                     "description": "Description for transaction_process_rank"
    //                                 },
    //                                 {
    //                                     "assetId": "connectorcreated",
    //                                     "edgeType": "ops",
    //                                     "graphName": "phoenix",
    //                                     "id": 71,
    //                                     "impactColor": 1,
    //                                     "level": 1,
    //                                     "description": "Description for connectorcreated"
    //                                 },
    //                                 {
    //                                     "assetId": "mds_level",
    //                                     "edgeType": "ops",
    //                                     "graphName": "phoenix",
    //                                     "id": 26,
    //                                     "impactColor": 2,
    //                                     "level": 1,
    //                                     "description": "Description for mds_level"
    //                                 }
    //                             ],
    //                             "edgeType": "",
    //                             "graphName": "phoenix",
    //                             "id": 1,
    //                             "impactColor": 0,
    //                             "level": 0,
    //                             "description": "Description for connector_last_checked"
    //                         }
    //                     ]
    //                 }
    //             ],
    //             "name": "levelTraverser"
    //         }
    //         // ... (other impactAnalyzer items)
    //     ]
    // };
    // elements.value = processData(rawData);
    searchNode();
});

onPaneReady(({ fitView }) => {
    fitView();
});
</script>

<style scoped>
.vue-flow-wrapper {
    width: 100%;
    height: 600px;
    background-color: #fff;
}

.custom-tooltip {
    position: fixed;
    background-color: rgba(0, 0, 0, 0.8);
    color: white;
    padding: 5px 10px;
    border-radius: 4px;
    font-size: 14px;
    z-index: 1000;
}
</style>