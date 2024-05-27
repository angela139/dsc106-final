<script>
import { onMount } from 'svelte';
import * as d3 from 'd3';

let hatecrime_data = [];

const numbers = [0, 1, 2, 3, 4, 5]

const categories = [
  'Disability', 'Gender', 'Gender Nonconforming', 
  'Race/Ethnicity/Ancestry', 'Religion', 'Sexual Orientation'
];

const terms = [
  "2001-2004: George W. Bush's 1st Term",
  "2005-2008: George W. Bush's 2nd Term",
  "2009-2012: Barack Obama's 1st Term",
  "2013-2016: Barack Obama's 2nd Term",
  "2017-2020: Donald Trump's Term",
  "2021-2024: Biden's Term"
]

let selectedTerm = terms[0];

const blurbs = ['9/11', 'idk', 'obamna', "what's obama's last name", 'guy ig', 'joever']

let selectedBlurb = blurbs[0];

onMount(() => {
    loadDataandChart();
});

const loadDataandChart = async() => {
  load_data().then(termData => {
      updateChart(termData);
  });
};

const load_data = () => {
  return new Promise((resolve, reject) => {
    const termData = {};

    d3.csv("./src/data/summed_victims_terms.csv").then(fullData => {
      const termData = fullData.map(d => ({
        most_serious_bias_type: d['most_serious_bias_type'],
        value: parseFloat(d[selectedTerm])  // Make sure selectedTerm corresponds to a column name in CSV
      }));
      console.log(termData);
      hatecrime_data = termData;
      resolve(termData);
    }).catch(error => {
      console.error("Error loading the CSV data:", error);
      reject(error);
    });
  });
};

const updateChart = async(data) => {
  const margin = { top: 20, right: 30, bottom: 30, left: 40 };
  const width = 500 - margin.left - margin.right;
  const height = 300 - margin.top - margin.bottom;
  console.log(`data: ${data}`)
  // Clear previous SVG if exists
  d3.select("#bar-chart").select("svg").remove();

  const svg = d3.select("#bar-chart")
                .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
                .append("g")
                .attr("transform", `translate(${margin.left},${margin.top})`);

  const x = d3.scaleBand()
    .range([0, width])
    .padding(0.1)
    .domain(data.map(d => d.most_serious_bias_type));

  const y = d3.scaleLinear()
    .range([height, 0])
    .domain([0, d3.max(data, d => d.value)]);

  svg.selectAll(".bar")
    .data(data)
    .enter().append("rect")
    .attr("class", "bar")
    .attr("x", d => x(d.most_serious_bias_type))
    .attr("width", x.bandwidth())
    .attr("y", d => y(d.value))
    .attr("height", d => height - y(d.value))
    .attr("fill", "steelblue");

  svg.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x));

  svg.append("g")
    .call(d3.axisLeft(y));
}

function selectTerm(num) {
  selectedTerm = terms[num];
  selectedBlurb = blurbs[num];
  console.log(selectedTerm);
  loadDataandChart();
}

</script>


<main>
  <h1>DSC 106 Final Project</h1>
  <div>
    <div id="bar-chart"></div>
    <div class="time-scale">
        {#each numbers as num}
          <button on:click={() => selectTerm(num)}>
              {terms[num]}
          </button>
        {/each}
    </div>
    <div class="blurb">{selectedBlurb}</div>
  </div>
  <p>Write your HTML here</p>
</main>

<style>
  .time-scale button {
    margin: 5px;
    padding: 5px;
  }
  .blurb {
    margin-top: 20px;
    padding: 10px;
    border: 1px solid #ccc;
  }

  .bar {
    fill: steelblue;
  }

  svg {
    background-color: #f4f4f4;
    border: 1px solid #ccc;
  }
</style>
