<script setup>
import { Panel, VueFlow, useVueFlow} from '@vue-flow/core'
import { ref } from 'vue'
import { Background, BackgroundVariant } from '@vue-flow/background'
import { MiniMap } from '@vue-flow/minimap'
import axios from 'axios';

const instance = useVueFlow()
const { nodes, addNodes, addEdges, 
  onConnect, dimensions, onEdgeClick } = useVueFlow()
const source = ref(null);
const target = ref(null);

onConnect((params) => {
  const edge = {
    source: params.source,
    target: params.target,
    markerEnd: 'arrow',
    label: '1',
    data: {
      s: params.source,
      t: params.target,
      cap: '1',
    },
    
  };

  addEdges([edge]);
});

onEdgeClick((event, edge) => {
  console.log(event);
  //console.log(edge);
  const edgeTemp = instance.findEdge(event.edge.id);
  console.log(edgeTemp);
  const newCap = prompt('Enter new capacity');
  if (newCap != null) {
    edgeTemp.data.cap = newCap;
    edgeTemp.label = newCap;
  }
}
);

async function sendRequest(){
  //check if source and target are not null
  if (source.value == null || target.value == null) {
    alert('Source or target is null');
    return;
  }
  //check if source and target are in the graph
  const sourceNode = instance.findNode(source.value);
  const targetNode = instance.findNode(target.value);
  if (sourceNode == null || targetNode == null) {
    alert('Source or target is not in the graph');
    return;
  }
  //check if source and target are not the same
  if (sourceNode.id == targetNode.id) {
    alert('Source and target are the same');
    return;
  }
  console.log(instance.edges);
  //check if there is atleast one edge
  if (instance.edges.value.length == 0) {
    alert('There is no edge');
    return;
  }
  
  let edgesList = [];

  for(let i = 0; i < instance.edges.value.length; i++){
    let edge = instance.edges.value[i];
    let edgeTemp = {
      s: edge.data.s,
      t: edge.data.t,
      c: edge.data.cap
    }
    edgesList.push(edgeTemp);
  };

  let data = {
    nodes: Number.parseInt(instance.nodes.value.length) ,
    noOfEdges: Number.parseInt(instance.edges.value.length),
    s: source.value,
    t: target.value,
    edges: edgesList    
}
          try {
            const response = await axios.post(
              "http://localhost:5002/api/calculateMaxFlow",data
            );
            // JSON responses are automatically parsed.
         //   this.posts = response.data;
         console.log(response.data);
          } catch (error) {
            console.log(error);
          }


}

function addRandomNode() {
  const nodeId = (nodes.value.length + 1).toString()

  const newNode = {
    id: nodeId,
    label: `Node: ${nodeId}`,
    position: { x: Math.random() * dimensions.value.width, y: Math.random() * dimensions.value.height },
  }

  addNodes([newNode])
}
</script>

<template>
  <VueFlow >
    <MiniMap />
    <template>
      
    </template>

    <Background :variant="BackgroundVariant.Lines" />

    <Panel position="top-right">
      <button type="button" @click="addRandomNode">add node</button>
      <input type="text" v-model="source" placeholder="source" />
      <input type="text" v-model="target" placeholder="target" />
      <button type="button" @click="sendRequest">Calculate MaxFlow</button>
    </Panel>
  </VueFlow>
</template>
