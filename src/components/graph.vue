<template>
  <div class="">
    <svg :width="width" :height="height" style="border: 1px green solid"></svg>
  </div>
</template>

<script>
import * as d3 from 'd3';
import friendCategoryNodes from '../data/friendCategoryNodes';
import friendCategory from '../data/friendNodes';
import friendLinks from '../data/friendLinks';

export default {
  data() {
    return {
      svg: null,
      node: null,
      label: null,
      nodes_data: [],
      links_data: [],
      radius: 7,
      width: 700,
      height: 500,
      // width: Math.max(document.documentElement.clientWidth, window.innerWidth || 0),
      // height: Math.max(document.documentElement.clientHeight, window.innerHeight || 0),
      simulation: null,
    };
  },
  computed: {
  },
  methods: {
    drawNode() {
      return this.node
        .attr('cx', n => Math.max(20, Math.min(this.width - 20, n.x)))
        .attr('cy', n => Math.max(20, Math.min(this.height - 40, n.y)));
    },
    drawLabels() {
      return this.label
        .attr('x', n => Math.max(20, Math.min(this.width - 20, n.x) + 10))
        .attr('y', n => Math.max(20, Math.min(this.height - 20, n.y) - 10));
    },
    drawLink() {
      return this.link
        .attr('x1', n => n.source.x)
        .attr('y1', n => n.source.y)
        .attr('x2', n => n.target.x)
        .attr('y2', n => n.target.y);
    },
    tickActions() {
      this.drawLabels();
      this.drawNode();
      this.drawLink();
    },
  },
  mounted() {
    this.nodes_data = [
      ...friendCategory.data,
      ...friendCategoryNodes.data,
    ];
    this.links_data = [
      ...friendLinks.data,
    ];
    this.svg = d3.select('svg');
    this.simulation = d3.forceSimulation()
      .nodes(this.nodes_data);
    // this.link_force = d3.forceLink(this.links_data)
    //   .id(d => d.id);
    this.simulation
      .force('charge_force', d3.forceManyBody().strength(() => -300).distanceMax(200))
      .force('center_force', d3.forceCenter(this.width / 2, this.height / 2))
      .force('links', d3.forceLink(this.links_data).id(d => d.id));

    this.node = this.svg.append('g')
      .attr('class', 'nodes')
      .selectAll('circle')
      .data(this.nodes_data)
      .enter()
      .append('circle')
      .attr('r', n => (n.type === 'core' ? this.radius : this.radius - 2))
      .attr('fill', n => (n.type === 'core' ? 'blue' : 'red'));

    this.label = this.svg.append('g')
      .attr('class', 'label')
      .selectAll('g')
      .data(this.nodes_data)
      .enter()
      .append('text')
      .text(n => n.name)
      .attr('font-size', '10')
      .attr('font-weight', n => (n.type === 'core' ? 'bold' : 'normal'));

    this.link = this.svg.append('g')
      .attr('class', 'links')
      .selectAll('line')
      .data(this.links_data)
      .enter()
      .append('line')
      .attr('stroke-width', 1)
      .attr('stroke', '#d3d3d3');

    this.simulation.on('tick', this.tickActions);
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
</style>
