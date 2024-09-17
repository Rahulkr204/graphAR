<template>
    <div class="vue-flow-wrapper">
        <div class="flex justify-between">
            <div class="text-4xl font-medium">Impact Analysis</div>
            <div class="flex flex-row">
                <input type="number" v-model="searchTerm" placeholder="Enter node number..."
                    @change="e => searchTerm = e.target.value" @keyup.enter="searchNode"
                    class="px-2 py-1 mr-2 bg-white border border-blue-400 border-solid rounded outline-none basis-11/12" />
                <!-- <button @click="searchNode"
                    class="w-full text-white bg-blue-500 border border-blue-500 border-solid rounded basis-1/12">Search</button> -->
            </div>
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

        <!-- Loader overlay -->
        <div v-if="isLoading" class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-75">
            <!-- <div class="w-16 h-16 border-t-4 border-blue-500 border-solid rounded-full animate-spin"></div> -->
            <!-- <div class="animate-spin inline-block size-6 border-[3px] border-current border-t-transparent text-blue-600 rounded-full"
                role="status" aria-label="loading">
                <span class="sr-only">Loading...</span>
            </div> -->
            <div class="relative">
                <div class="p-4 border border-blue-200 rounded-lg bg-blue-50">
                    <div class="flex">
                        <div class="shrink-0">
                            <svg class="shrink-0 size-4 text-blue-600 mt-0.5" xmlns="http://www.w3.org/2000/svg" width="24"
                                height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
                                stroke-linecap="round" stroke-linejoin="round">
                                <path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3Z"></path>
                                <path d="M12 9v4"></path>
                                <path d="M12 17h.01"></path>
                            </svg>
                        </div>
                        <div class="ms-3">
                            <div class="mt-2 text-sm text-blue-700">
                                <p v-if="loadingStep >= 1" class="font-semibold">Loading assets...</p>
                                <p v-if="loadingStep >= 2">Please wait while we jazz things up!</p>
                                <p v-if="loadingStep >= 3">Fetching the coolest nodes and edges for you...</p>
                                <p v-if="loadingStep >= 4">Almost there! Preparing the graph magic...</p>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="absolute top-0 rounded-lg start-0 size-full bg-white/50"></div>

                <div class="absolute transform -translate-x-1/2 -translate-y-1/2 top-1/2 start-1/2">
                    <div class="animate-spin inline-block size-6 border-[3px] border-current border-t-transparent text-blue-600 rounded-full"
                        role="status" aria-label="loading">
                        <span class="sr-only">Loading...</span>
                    </div>
                </div>
            </div>
        </div>
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

const isLoading = ref(false);
const loadingStep = ref(0);

const searchNode = async () => {
    isLoading.value = true;
    loadingStep.value = 0;

    try {
        loadingStep.value = 1;
        await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate delay

        loadingStep.value = 2;
        const response = await fetch(`https://c10d-52-66-4-2.ngrok-free.app/api/v1/report/atlan/Table/impact_analysis_report?internalID=${searchTerm.value || 1}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                "change_type": "ADD_NEW_FIELD",
                "version": "1.0.1",
            })
        });

        loadingStep.value = 3;
        await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate delay

        const data = await response.json();

        loadingStep.value = 4;
        await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate delay

        elements.value = processData(data);
    } catch (error) {
        console.error('Error fetching data:', error);
    } finally {
        isLoading.value = false;
        loadingStep.value = 0;
    }
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
                        title: connection.description || 'Description not available',
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
    await searchNode();
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
    position: relative;
    /* Add this to allow absolute positioning of the loader */
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