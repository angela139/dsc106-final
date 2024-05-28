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

const blurbs = ['On September 11, 2001, terrorists from the Islamic extremist group Al Qaeda hijacked planes and crashed them into the World Trade Center Towers in New York City. Due to the terrorist attacks, almost 3,000 people lost their lives. However, after these attacks, there were concerns of anti-Muslim hate in the United States. The then-president George W. Bush would try to ease tensions by making a speech in the Islamic Center in Washington D.C. which worked in the beginning with the majority of Americans sharing a positive view of Muslims in November 2001. However, these sentiments would not last long. According to the Pew Research Center, “28% of adults said they had grown more suspicious of people of Middle Eastern descent” in September 2001 which “grew to 36% less than a year later” (Hartig & Doherty, 2021). In the next few years, they noted that Republicans were much more likely to associate Muslims with committing violence than Democrats.', 
'During the George W. Bush’s second term, the United States began to grapple with issues with the anti-LGBTQ+ hate. On September 6, 2005, for the first time, the California Legislature passed a bill to allow same-sex marriage but then-governor Arnold Schwarzenegger vetoes it (CNN, 2024). In a survey conducted by Harris Interactive in 2006, they found that 54% of LGBTQ+ people were worried about being hate crimed. Nonetheless, the public sentiment towards LGBTQ+ wouldn’t improve much in the next few years. In 2008, California voters would pass Proposition 8 which would make same-sex marriage illegal.', 
'On October 28, 2009, newly-elected President Barack Obama signed Matthew Shepard and James Byrd, Jr. Hate Crimes Prevention Act into law which made it a federal crime for someone to assault another person based on their gender identity or sexual orientation. This landmark legislation was a significant achievement for the LGBTQ+ rights movement, providing legal protection against violence. In the next few years, more strides were made to protect their rights including Proposition 8 being labeled unconstitutional in 2010 by a federal judge. In 2012, President Obama would become the first president to publicly support same-sex marriage.', 
"While the LGBTQ+ movement was taking big steps to fight for their rights, another civil rights movement began to stir in 2013 and 2014. In 2013, Alicia Garza, Patrisse Cullors, and Opal Tometi formed the Black Lives Matter Movement in response to the murderer of African American teenager Trayvon Martin being acquitted. They gained media attention through #BlackLivesMatter in an effort to fight against injustice for black people. In 2014, there were notable cases of police brutality against African Americans. Another black teenager, Micheal Brown, lost his life after being shot by a police officer in Ferguson, Missouri. This caused #BlackLivesMatter to blow up on social media, with 1.7 million instances recorded three weeks after the officer was not indicted for the shooting according to World Economic Forum (Whiting, 2022).", 
'When Donald Trump began his term as president in 2017, his administration aimed to crack down on immigration policies, specifically targeting undocumented immigrants. He signed an executive order to limit travelers from Muslim countries such as Iran and Iraq. He would also focus his efforts on building the wall between the United States and Mexico border, determined to keep undocumented immigrants from Mexico from entering the country. Towards the end of his presidency in 2020, the coronavirus spread in the United States which led to Trump enacting a travel ban from China. Due to the pandemic, he also passed Title 42 which allowed him to keep migrants and asylum seekers on the border from entering the country. The Black Lives Matter movement would also have a resurgence in 2020 with the passing of George Floyd. On May 25, 2020, a black man, George Floyd, was pinned to death by police officers during an arrest. After the recording of his death was posted to the media, hundreds of protests erupted around the United States to fight against police brutality and racism. As reported by the New York Times, the protests saw a massive turnout, peaking at half a million participants on June 6, 2020 (Buchanan et al, 2020).', 
'During the rise of the coronavirus pandemic, Asian Americans faced hate crimes due to belief that they caused the pandemic. From March 2020 to March 2021, the organization Stop AAPI Hate reported that there were 6,603 hate crimes against Asian Americans, according to NPR. On May 20, 2021, President Biden signed the COVID-19 Hate Crimes Act in response to the rise of Asian American hate, which aimed to make reporting hate crimes easier. In the present day, the Palestine-Israel conflict has been gaining traction in the United States media, as people are protesting against the war. In universities such as Brown and UCLA, there have been student protests through tent encampments and walkouts to pressure the university heads to diverge their funds from supporting the war efforts. ']

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
  const margin = { top: 20, right: 30, bottom: 30, left: 60 };
  const width = 800;
  const height = 500;
  console.log(`data: ${data}`)
  // Clear previous SVG if exists
  d3.select("#bar-chart").select("svg").remove();

  const svg = d3.select("#bar-chart")
                .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom + 20)
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

  // Add x-axis label
  svg.append("text")
    .attr("text-anchor", "middle")
    .attr("x", width / 2)
    .attr("y", height + margin.bottom + 10)
    .text("Bias Type");

  // Add y-axis label
  svg.append("text")
    .attr("text-anchor", "middle")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", -margin.left + 20)
    .text("Number of Hate Crimes");
}

function selectTerm(num) {
  selectedTerm = terms[num];
  selectedBlurb = blurbs[num];
  console.log(selectedTerm);
  loadDataandChart();
}

</script>


<main>
  <h1>Examining the Link: The Rise of Hate Crimes And Social Justice Movements In San Francisco</h1>
  <div id="bar-chart"></div>
  <div class="time-scale">
      {#each numbers as num}
        <button on:click={() => selectTerm(num)}>
            {terms[num]}
        </button>
      {/each}
  </div>
  <p class="blurb">{selectedBlurb}</p>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .time-scale button {
    margin: 5px;
    padding: 5px;
  }
  .blurb {
    width: 90%;
    margin-top: 20px;
    padding: 20px;
    border: 1px solid #ccc;
    font-size: 18px;
  }

  .bar {
    fill: steelblue;
  }

  svg {
    background-color: #f4f4f4;
    border: 1px solid #ccc;
  }
</style>
