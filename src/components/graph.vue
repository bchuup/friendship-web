<template>
  <div class="">
    <svg :width="width" :height="height"></svg>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  data() {
    return {
      svg: null,
      node: null,
      label: null,
      nodes_data: [
        { id: 'house', name: 'House', type: 'core' },
        { id: 'chestnut', name: 'Chestnut', type: 'core' },
        { id: 'mph', name: 'MPH', type: 'core' },
        { id: 'business', name: 'Business School', type: 'core' },
        { id: 'soccer', name: 'Soccer', type: 'core' },
        { id: 'softball', name: 'Softball', type: 'core' },
        { id: 'ben_chu', name: 'Ben', type: 'person' },
        { id: 'leaticia_kaggwa', name: 'Teesha', type: 'person' },
        { id: 'lisa_snider', name: 'Lisa', type: 'person' },
        { id: 'pranai_vasudev', name: 'Pranai', type: 'person' },
        { id: 'grayson_ingraham', name: 'Grayson', type: 'person' },
        { id: 'rivka_kushner', name: 'Rivka', type: 'person' },
      ],
      links_data: [
        { source: 'chestnut', target: 'ben_chu' },
        { source: 'chestnut', target: 'leaticia_kaggwa' },
        { source: 'chestnut', target: 'lisa_snider' },
        { source: 'chestnut', target: 'pranai_vasudev' },
        { source: 'pranai_vasudev', target: 'grayson_ingraham' },
        { source: 'mph', target: 'rivka_kushner' },
        { source: 'ben_chu', target: 'rivka_kushner' },
        { source: 'rivka_kushner', target: 'grayson_ingraham' },
      ],
      width: 960,
      height: 600,
      simulation: null,
    };
  },
  computed: {
  },
  methods: {
    drawNode() {
      return this.node
        .attr('cx', n => n.x)
        .attr('cy', n => n.y);
    },
    drawLabels() {
      return this.label
        .attr('x', n => n.x - 10)
        .attr('y', n => n.y + 20);
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
    this.svg = d3.select('svg');
    this.simulation = d3.forceSimulation()
      .nodes(this.nodes_data);
    this.link_force = d3.forceLink(this.links_data)
      .id(d => d.id);
    this.simulation
      .force('charge_force', d3.forceManyBody().strength(() => -200).distanceMax(200))
      .force('center_force', d3.forceCenter(this.width / 2, this.height / 2))
      .force('links', this.link_force);

    this.node = this.svg.append('g')
      .attr('class', 'nodes')
      .selectAll('circle')
      .data(this.nodes_data)
      .enter()
      .append('circle')
      .attr('r', n => (n.type === 'core' ? 7 : 5))
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
