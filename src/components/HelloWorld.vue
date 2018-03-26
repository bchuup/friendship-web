<template>
  <svg xmlns="http://www.w3.org/2000/svg"
    :width="width+'px'"
    :height="height+'px'"
    @mousemove="drag($event)"
    @mouseup="drop()"
    v-if="bounds.minX">

    <line
      v-for="(link, index) in graph.links"
      :key="`line-${index}`"
      :x1="coords[link.source.index].x"
      :y1="coords[link.source.index].y"
      :x2="coords[link.target.index].x"
      :y2="coords[link.target.index].y"
      stroke="black"
      stroke-width="2"/>

    <circle
      v-for="(node, i) in graph.nodes"
      :key="`nodes-${i}`"
      :cx="coords[i].x"
      :cy="coords[i].y"
      :r="20"
      :fill="colors[Math.ceil(Math.sqrt(node.index))]"
      stroke="white"
      stroke-width="1"
      @mousedown="currentMove = {x: $event.screenX, y: $event.screenY, node: node}"/>
  </svg>
</template>

<script>
import * as d3 from 'd3';

export default {
  data() {
    return {
      graph: {
        nodes: d3.range(10).map(i => ({ index: i, x: 0, y: 0 })),
        links: d3.range(9).map(i => ({ source: Math.floor(Math.sqrt(i)), target: i + 1 })),
      },
      width: Math.max(document.documentElement.clientWidth, window.innerWidth || 0),
      height: Math.max(document.documentElement.clientHeight, window.innerHeight || 0) - 40,
      padding: 20,
      colors: ['#2196F3', '#E91E63', '#7E57C2', '#009688', '#00BCD4', '#EF6C00', '#4CAF50', '#FF9800', '#F44336', '#CDDC39', '#9C27B0'],
      simulation: null,
      currentMove: null,
    };
  },
  computed: {
    bounds() {
      return {
        minX: Math.min(...this.graph.nodes.map(n => n.x)),
        maxX: Math.max(...this.graph.nodes.map(n => n.x)),
        minY: Math.min(...this.graph.nodes.map(n => n.y)),
        maxY: Math.max(...this.graph.nodes.map(n => n.y)),
      };
    },
    coords() {
      const xDenom = this.bounds.maxX - this.bounds.minX;
      const yDenom = this.bounds.maxY - this.bounds.minY;
      const nodes = this.graph.nodes.map((node) => {
        const xNumerator = (node.x - this.bounds.minX) * (this.width - (2 * this.padding));
        const yNumerator = (node.y - this.bounds.minY) * (this.height - (2 * this.padding));
        return {
          x: this.padding + (xNumerator / xDenom),
          y: this.padding + (yNumerator / yDenom),
        };
      });
      return nodes;
    },
  },
  created() {
    this.simulation = d3.forceSimulation(this.graph.nodes)
      .force('charge', d3.forceManyBody().strength(() => -100))
      .force('link', d3.forceLink(this.graph.links))
      .force('x', d3.forceX())
      .force('y', d3.forceY());
  },
  methods: {
    drag(e) {
      if (this.currentMove) {
        this.currentMove.node.fx = this.currentMove.node.x -
          (((this.currentMove.x - e.screenX) * (this.bounds.maxX - this.bounds.minX)) /
            (this.width - (2 * this.padding)));
        this.currentMove.node.fy = this.currentMove.node.y -
          (((this.currentMove.y - e.screenY) * (this.bounds.maxY - this.bounds.minY)) /
            (this.height - (2 * this.padding)));
        this.currentMove.x = e.screenX;
        this.currentMove.y = e.screenY;
      }
    },
    drop() {
      if (this.currentMove) {
        delete this.currentMove.node.fx;
        delete this.currentMove.node.fy;
      }
      this.currentMove = null;
      this.simulation.alpha(1);
      this.simulation.restart();
    },
  },
  mounted() {
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="styl" scoped>
  @import '~styl/colors.styl';

  .links line {
    stroke: #999;
    stroke-opacity: 0.6;
  }

  .nodes circle {
    stroke: #fff;
    stroke-width: 1.5px;
  }

  h1, h2 {
    font-weight: normal;
    color: $green;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
  }
</style>
