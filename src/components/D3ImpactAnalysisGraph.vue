<template>
    <div>
        <div v-if="loading">Loading...</div>
        <div v-else-if="error">Error: {{ error }}</div>
        <div v-else ref="chartContainer" class="chart-container"></div>
        <div>Nodes: {{ nodeCount }}, Links: {{ linkCount }}</div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import * as d3 from 'd3';

const chartContainer = ref(null);
const loading = ref(true);
const error = ref(null);
const nodeCount = ref(0);
const linkCount = ref(0);

const width = 1200;
const height = 800;

const impactColorMap = {
    0: '#000000',
    1: '#0000FF',
    2: '#FFA500',
    3: '#FF0000'
};

const processData = (data) => {
    const nodes = new Map();
    const links = [];

    data.impactAnalyzer.forEach((analyzer, analyzerIndex) => {
        analyzer.connectedVertices.forEach(level => {
            level.levelConnectedVertices.forEach(vertex => {
                const nodeId = `${analyzerIndex}-${vertex.id}`;
                nodes.set(nodeId, {
                    id: nodeId,
                    name: vertex.assetId,
                    level: vertex.level
                });

                vertex.connections.forEach(connection => {
                    const targetId = `${analyzerIndex}-${connection.id}`;
                    if (connection.impactColor !== 0) {
                        links.push({
                            source: nodeId,
                            target: targetId,
                            color: impactColorMap[connection.impactColor]
                        });
                    }
                });
            });
        });
    });

    return { nodes: Array.from(nodes.values()), links };
};

const createGraph = (data) => {
    const svg = d3.select(chartContainer.value)
        .append('svg')
        .attr('width', width)
        .attr('height', height);

    // Filter out links with missing nodes
    const validLinks = data.links.filter(link =>
        data.nodes.some(node => node.id === link.source) &&
        data.nodes.some(node => node.id === link.target)
    );

    const simulation = d3.forceSimulation(data.nodes)
        .force('link', d3.forceLink(validLinks).id(d => d.id).distance(100))
        .force('charge', d3.forceManyBody().strength(-300))
        .force('center', d3.forceCenter(width / 2, height / 2))
        .force('collision', d3.forceCollide().radius(50));

    const link = svg.append('g')
        .selectAll('line')
        .data(validLinks)
        .join('line')
        .attr('stroke', d => d.color)
        .attr('stroke-width', 2);

    const node = svg.append('g')
        .selectAll('g')
        .data(data.nodes)
        .join('g');

    node.append('circle')
        .attr('r', 20)
        .attr('fill', d => d.level === 0 ? '#ff9999' : '#99ccff');

    node.append('text')
        .text(d => d.name)
        .attr('x', 25)
        .attr('y', 5)
        .attr('font-size', '12px');

    node.append('title')
        .text(d => d.name);

    simulation.on('tick', () => {
        link
            .attr('x1', d => d.source.x)
            .attr('y1', d => d.source.y)
            .attr('x2', d => d.target.x)
            .attr('y2', d => d.target.y);

        node
            .attr('transform', d => `translate(${d.x},${d.y})`);
    });

    // Add zoom capabilities
    const zoom = d3.zoom()
        .on('zoom', (event) => {
            svg.selectAll('g').attr('transform', event.transform);
        });

    svg.call(zoom);

    nodeCount.value = data.nodes.length;
    linkCount.value = validLinks.length;
};

onMounted(async () => {
    try {
        // For testing, we'll use the data directly instead of fetching
        const rawData = {
            "impactAnalyzer": [
                {
                    "connectedVertices": [
                        {
                            "level": 0,
                            "levelConnectedVertices": [
                                {
                                    "assetId": "connector_last_checked",
                                    "connections": [
                                        {
                                            "assetId": "_fivetran_job_last_checked",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 12,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "transaction_process_rank",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 37,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connector_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 15,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "process_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 16,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "customer_name",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 17,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connectorcreated",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 71,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "product_public",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 22,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "event_deleted",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 23,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_subtotal",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 25,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_level",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 26,
                                            "impactColor": 2,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connectorlast_login",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 27,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "Snowflake_public",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 28,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "job_valid",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 30,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_alias",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 31,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "dbt_level",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 32,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "transaction_process_amount",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 36,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "Snowflake_deleted",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 39,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "job_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 41,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "transaction_process_date",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 43,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "destination_dest_hidden",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 44,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "user_phone",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 45,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "shipment_last_updated",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 46,
                                            "impactColor": 2,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_created",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 47,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_valid",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 49,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "dbt_quantity",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 9,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "order_enabled",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 50498,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "destination_dest_level",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 52096,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_description",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 71245,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "product_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 74556,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "event_number",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 43083,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "job_quantity",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 31638,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "job_last_seen",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 9956,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "destination_dest_subtotal",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 11,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "urn_common_last_modified",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 2,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "invoice_total",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 3,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "product_external",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 6,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "pipe_hidden",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 7,
                                            "impactColor": 0,
                                            "level": 1
                                        }
                                    ],
                                    "edgeType": "",
                                    "graphName": "phoenix",
                                    "id": 1,
                                    "impactColor": 0,
                                    "level": 0
                                }
                            ]
                        }
                    ],
                    "name": "levelTraverser"
                },
                {
                    "connectedVertices": [
                        {
                            "level": 0,
                            "levelConnectedVertices": [
                                {
                                    "assetId": "urn_common_last_modified",
                                    "connections": [
                                        {
                                            "assetId": "mds_subtotal",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 25,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "event_deleted",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 23,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_level",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 26,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connectorlast_login",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 27,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "Snowflake_public",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 28,
                                            "impactColor": 3,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "job_valid",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 30,
                                            "impactColor": 2,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "product_public",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 22,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "dbt_quantity",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 9,
                                            "impactColor": 2,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "backup_inactive",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 20,
                                            "impactColor": 3,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_alias",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 31,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "dbt_level",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 32,
                                            "impactColor": 2,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "transaction_process_amount",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 36,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "transaction_process_rank",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 37,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "shipment_level",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 38,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "dbt_locked",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 40,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connector_alias",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 42,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "destination_dest_hidden",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 44,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "user_phone",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 45,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "shipment_last_updated",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 46,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "invoice_last_checked",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 48,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "calls_hidden",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 51,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "process_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 16,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "order_email",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 95545,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "_fivetran_job_price",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 94653,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "source_last_checked",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 93086,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "calls_price",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 34533,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "user_private",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 21494,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "urn_common_last_checked",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 4,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "product_external",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 6,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "pipe_hidden",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 7,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "dbt_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 5,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "destination_dest_subtotal",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 11,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "_fivetran_job_last_checked",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 12,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connector_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 15,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "customer_name",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 17,
                                            "impactColor": 1,
                                            "level": 1
                                        }
                                    ],
                                    "edgeType": "",
                                    "graphName": "phoenix",
                                    "id": 2,
                                    "impactColor": 0,
                                    "level": 0
                                }
                            ]
                        }
                    ],
                    "name": "levelTraverser"
                },
                {
                    "connectedVertices": [
                        {
                            "level": 0,
                            "levelConnectedVertices": [
                                {
                                    "assetId": "dbt_level",
                                    "connections": [
                                        {
                                            "assetId": "source_deleted",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 77,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "shipment_last_updated",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 46,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connector_price",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 79,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "user_active",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 57,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "_fivetran_sync_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 56,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "process_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 54,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "event_internal",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 53,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "invoice_internal",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 50,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "mds_created",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 47,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "invoice_amount",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 76,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "src_total",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 75,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "invoice_id",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 74,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "user_price",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 73,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connectorcreated",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 71,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "product_inactive",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 69,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "destination_dest_last_login",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 68,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "process_count",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 67,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "process_quantity",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 63,
                                            "impactColor": 2,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "_fivetran_job_alternate",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 62,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connector_last_login",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 61,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "order_price",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 59,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "event_deleted",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 58,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "job_step",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 41,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "_fivetran_job_inactive",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 75102,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "user_phone",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 45,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "Snowflake_last_seen",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 6684,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "event_valid",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 21790,
                                            "impactColor": 2,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "_fivetran_sync_last_accessed",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 36198,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "product_phone",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 58835,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connectoralias",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 69048,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "source_last_updated",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 91832,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "transaction_process_rank",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 37,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "connector_alias",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 42,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "Snowflake_deleted",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 39,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "invoice_quantity",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 35,
                                            "impactColor": 1,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "pipe_hidden",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 34,
                                            "impactColor": 0,
                                            "level": 1
                                        },
                                        {
                                            "assetId": "source_public",
                                            "edgeType": "ops",
                                            "graphName": "phoenix",
                                            "id": 33,
                                            "impactColor": 1,
                                            "level": 1
                                        }
                                    ],
                                    "edgeType": "",
                                    "graphName": "phoenix",
                                    "id": 32,
                                    "impactColor": 0,
                                    "level": 0
                                }
                            ]
                        }
                    ],
                    "name": "levelTraverser"
                }
            ]
        }
        const processedData = processData(rawData);
        createGraph(processedData);
        loading.value = false;
    } catch (e) {
        error.value = e.message;
        loading.value = false;
        console.error("Error processing data:", e);
    }
});

onUnmounted(() => {
    // Clean up D3 elements if necessary
    d3.select(chartContainer.value).selectAll('*').remove();
});
</script>

<style scoped>
.chart-container {
    width: 100%;
    height: 800px;
    border: 1px solid #ccc;
}
</style>