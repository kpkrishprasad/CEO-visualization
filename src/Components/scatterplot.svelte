<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  
  let ceoData = [];
  onMount(async () => {
    const response = await fetch('/CEO-visualization/ceo_data_b.json');
    ceoData = await response.json();

    const industries = Array.from(new Set(ceoData.map(d => d.Industry)));
    const colorScale = d3.scaleOrdinal(d3.schemeCategory10).domain(industries);

    const margin = {top: 50, right: 300, bottom: 50, left: 60},
          width = 1000 - margin.left - margin.right,
          height = 600 - margin.top - margin.bottom;

    const svg = d3.select("#scatterplot")
                  .append("svg")
                  .attr("width", width + margin.left + margin.right)
                  .attr("height", height + margin.top + margin.bottom)
                  .append("g")
                  .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    const x = d3.scaleLinear()
                .domain(d3.extent(ceoData, d => d.employeePay))
                .range([ 0, width ]);
    svg.append("g")
       .attr("transform", "translate(0," + height + ")")
       .call(d3.axisBottom(x));

    const y = d3.scaleLinear()
                .domain([5000000, 200000000])
                .range([ height, 0]);
    svg.append("g")
       .call(d3.axisLeft(y));
    const tooltip = d3.select("#scatterplot")
                      .append("div")
                      .attr("class", "tooltip")
                      .style("opacity", 0);

    svg.append("g")
       .selectAll("dot")
       .data(ceoData)
       .enter()
       .append("circle")
       .attr("cx", d => x(d.employeePay))
       .attr("cy", d => y(d.ceoPay))
       .attr("r", 5)
       .style("fill", d => colorScale(d.Industry))
       .on("mouseover", (event, d) => {
         tooltip.html(`Industry: ${d.Industry}<br/>Company: ${d.company}<br/>CEO: ${d.ceo}<br/>CEO Pay: ${d.ceoPay}<br/>Median Worker Pay: ${d.employeePay}`)
                .style("opacity", 1)
                .style("left", (event.pageX + 10) + "px")
                .style("top", (event.pageY + 10) + "px");
       })
       .on("mouseleave", () => {
         tooltip.style("opacity", 0);
       });
  });
</script>

<style>
  .tooltip {
    position: absolute;
    text-align: left;
    width: auto;
    height: auto;
    padding: 8px;
    font: 12px sans-serif;
    background: lightsteelblue;
    border: 0px;
    border-radius: 8px;
    pointer-events: none;
  }
</style>

<div class="user-instructions">
  <p>
  Hover over the points to see details. Click on a point to highlight. Zoom in/out.
  </p>
</div>
<div id="scatterplot"></div>
<div class="graph explanation">
  <p>
    The CEO Pay Ratio data ranges from a minimum of 0 at PACCAR to a maximum of 4628 at Western Digital. The pay ratio measures how many times more the CEO is paid compared to the median worker within the same company. For instance, at Western Digital, the CEO, David V. Goeckeler, earns 4628 times the median employee pay. This graph provides a visual representation of the disparity in compensation between top executives and average workers across various industries, highlighting significant differences and trends. It reveals, for example, how sectors like Technology tend to have higher discrepancies, reflecting broader economic and social dynamics within these industries.
  </p>
</div>
