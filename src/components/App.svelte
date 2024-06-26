<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { base } from '$app/paths';

  let currentSection = -2;
  let innerWidth;

  const terms = [
    "Overview",
    "2001-2004: George W. Bush's 1st Term",
    "2005-2008: George W. Bush's 2nd Term",
    "2009-2012: Barack Obama's 1st Term",
    "2013-2016: Barack Obama's 2nd Term",
    "2017-2020: Donald Trump's Term",
    "2021-2024: Biden's Term"
  ];

  const blurbs = [
    '\tIn recent years, we’ve seen a rapid rise in social justice movements and increased political polarization.\
     As we step into another presidential term this election year, we take a look at historical trends of how past \
     presidents\' actions may have influenced hate crimes, with San Francisco, a specifically diverse city, under \
     the microscope.\n\tKeep scrolling to dive deeper with us.',
    '\tOn September 11, 2001, terrorists from the Islamic extremist group Al Qaeda hijacked planes and crashed \
    them into the World Trade Center Towers in New York City. Due to the terrorist attacks, almost 3,000 people \
    lost their lives. However, after these attacks, there were concerns of anti-Muslim hate in the United States. \
    The then-president George W. Bush would try to ease tensions by making a speech in the Islamic Center in \
    Washington D.C. which worked in the beginning with the majority of Americans sharing a positive view of \
    Muslims in November 2001. However, these sentiments would not last long. According to the Pew Research \
    Center, “28% of adults said they had grown more suspicious of people of Middle Eastern descent” in September \
    2001 which “grew to 36% less than a year later” (Hartig & Doherty, 2021).',
    '\tDuring George W. Bush’s second term, the United States began to grapple with issues with anti-LGBTQ+ hate. \
    On September 6, 2005, for the first time, the California Legislature passed a bill to allow same-sex marriage \
    but then-governor Arnold Schwarzenegger vetoed it (CNN, 2024). In a survey conducted by Harris Interactive in \
    2006, they found that 54% of LGBTQ+ people were worried about being victims of hate crimes. Nonetheless, the \
    public sentiment towards LGBTQ+ wouldn’t improve much in the next few years. In 2008, California voters passed \
    Proposition 8 which would make same-sex marriage illegal. ',
    'On October 28, 2009, newly-elected President Barack Obama signed the Matthew Shepard and James Byrd, Jr. Hate \
    Crimes Prevention Act into law which made it a federal crime for someone to assault another person based on \
    their gender identity or sexual orientation. This landmark legislation was a significant achievement for the \
    LGBTQ+ rights movement, providing legal protection against violence. In the next few years, more strides were \
    made to protect their rights including Proposition 8 being labeled unconstitutional in 2010 by a federal judge. \
    In 2012, President Obama became the first president to publicly support same-sex marriage. ',
    '\tThe LGBTQ+ movement took big steps in their fight for rights with the Supreme Court ruling to protect gay \
    marriage. However, another civil rights movement began to stir in 2013 and 2014. In 2013, Alicia Garza, \
    Patrisse Cullors, and Opal Tometi formed the Black Lives Matter Movement in response to George Zimmerman, \
    the murderer of African American teenager Trayvon Martin, being acquitted. They gained media attention \
    through #BlackLivesMatter in an effort to fight against injustice for black people. In 2014, there were \
    notable cases of police brutality against African Americans. Another black teenager, Micheal Brown, lost \
    his life after being shot by a police officer in Ferguson, Missouri. This caused #BlackLivesMatter to blow \
    up on social media, with 1.7 million instances recorded three weeks after the officer was not indicted for \
    the shooting according to World Economic Forum (Whiting, 2022). With this social movement, there was pushback \
    in the form of the #AllLivesMatter movement. ',
    '\tWhen Donald Trump began his term as president in 2017, his administration aimed to crack down on immigration \
    policies, specifically targeting undocumented immigrants. He signed an executive order to limit travelers from \
    Muslim countries such as Iran and Iraq. He would also focus his efforts on building the wall between the United \
    States and Mexico border, determined to keep undocumented immigrants from Mexico from entering the country. \
    Towards the end of his presidency in 2020, COVID-19 hit the United States which led to Trump enacting a \
    travel ban from China. Due to the pandemic, he also passed Title 42 which allowed him to keep migrants and \
    asylum seekers on the border from entering the country. \
    \n\tThe Black Lives Matter movement would also have an enormous resurgence in 2020 with the passing of George Floyd. On May 25, 2020, Floyd, a black man, was pinned to death by police officers during an arrest. After the recording of his death was posted to the media, hundreds of protests erupted around the United States to fight against police brutality and racism. As reported by the New York Times, the protests saw a massive turnout, peaking at half a million participants on June 6, 2020 (Buchanan et al, 2020). At the same time we saw counter movements also emerge, with conflicts erupting in violence like with the Kyle Rittenhouse case.',
    '\tDuring the rise of the coronavirus pandemic, Asian Americans faced hate crimes due to the belief that they caused the pandemic, exacerbated by rhetoric from former President Donald Trump who called COVID-19 the “China Virus”. From March 2020 to March 2021, the organization Stop AAPI Hate reported that there were 6,603 hate crimes against Asian Americans, according to NPR. On May 20, 2021, President Biden signed the COVID-19 Hate Crimes Act in response to the rise of Asian American hate, which aimed to make reporting hate crimes easier.\t\n In the present day, the Palestine-Israel conflict has been gaining traction in the United States media, as people are protesting against the war. With such a contested topic, there have been many protests on either side. Unfortunately, there has been a suspected rise in religious hate crimes since the start of the war, with anti-semitic and Islamophobic crimes targeting members on both sides of the conflict.'
  ];

  const images = ['https://images.unsplash.com/photo-1493997575474-58d1db687da3?q=80&w=2340&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D',
  'https://images.wsj.net/im-394615?width=1920',
  'https://static01.nyt.com/images/2009/02/08/business/08stream_600.jpg?quality=75&auto=webp',
  'https://www.matthewshepard.org/wp-content/uploads/2020/04/Obama-passing-prevention-act-1.jpg',
  'https://assets.apnews.com/05/85/8194b8d0be0147eb15649784e23a/04c53adcfa5341a29da266a6aba12d66',
  'https://static.politico.com/70/8e/53affa2c466695e48a9063551bda/20210112-trump-wall-ap-773.jpg',
  'https://media-cldnry.s-nbcnews.com/image/upload/t_fit-1240w,f_auto,q_auto:best/rockcms/2021-04/210413-Bellevue-stop-aapi-hate-rally-ac-424p-4e6a6a.jpg'
  ]

  onMount(() => {
    window.addEventListener('scroll', handleScroll);
    loadDataAndChart(terms[1]); // Load initial data
    numOverTime();
    document.getElementById("bias-selector").addEventListener("change", () => {
      numOverTime();
    });
    document.getElementById("bias-type-select").addEventListener("change", () => {
      loadBiasDataAndBarGraph();
    });
    handleScroll()
  });

  function handleScroll() {
    const sectionHeight = window.innerHeight;
    const docHeight = d3.select('.right').node().getBoundingClientRect().height;
    const newSection = Math.min(terms.length, Math.floor(window.scrollY / sectionHeight) - 2);

    if (currentSection == -2){
      d3.select('#line-chart').style("display", "block");
      d3.select('#line-chart').style("opacity", (window.scrollY / sectionHeight));
      d3.select('#bias-select').style("display", "inline");
      d3.select('#bias-select').style("opacity", (window.scrollY / sectionHeight));
      d3.select('#cover-img').style("display", "block");
      d3.select('#cover-img').style("opacity", 1-(window.scrollY / sectionHeight));
    }
    if (currentSection == -1){
      d3.select('#line-chart').style("display", "block")
      d3.select('#line-chart').style("opacity", (window.scrollY / sectionHeight));
      d3.select('#bias-select').style("display", "inline")
      d3.select('#bias-select').style("opacity", (window.scrollY / sectionHeight));
      d3.select('#cover-img').style("display", "none");
    }
    if(currentSection == 0){
      loadDataAndChart(terms[currentSection+1]);
    }
    if(currentSection == 6){
      d3.select('#bar-chart').style("display", "block");
    }
    if(currentSection == 7){
      d3.select('#line-chart').style("display", "none");
      d3.select('#bias-select').style("display", "none");
      d3.select('#bar-chart').style("display", "none");
      d3.select('#cover-img').style("display", "block");
      d3.select('#cover-img').style("opacity", 1-(docHeight-window.scrollY-883)/300);
      console.log((docHeight-window.scrollY-883));
    }
    if (newSection !== currentSection) {
      currentSection = newSection;
      if (currentSection < 6) {
        if(currentSection > -1){
          d3.select('#line-chart').style("display", "none");
          d3.select('#bias-select').style("display", "none");
          d3.select('#cover-img').style("display", "none");
        }
        loadDataAndChart(terms[currentSection+1]);
      }
    }
  }

  async function loadDataAndChart(selectedTerm) {
    const data = await load_data(selectedTerm);
    drawBar(data);
  }

  async function loadBiasDataAndBarGraph() {
    const selectedBias = document.getElementById("bias-type-select").value;
    const bias_data = await load_bias_data(selectedBias);
    updateChart(bias_data);
  }

  async function load_bias_data(selectedBias){
    const response = await fetch(`${base}/bias_type.csv`);
    const data = await response.text();
    const parsedData = d3.csvParse(data, d => ({
      most_serious_bias: d['most_serious_bias'],
      value: parseFloat(d[selectedBias])
    }));
    const nonZeroData = parsedData.filter(d => d.value !== 0);
    return nonZeroData;
  }

  async function load_data(selectedTerm) {
    if(selectedTerm == 0) selectedTerm = 1;
    const response = await fetch(`${base}/summed_victims_terms.csv`);
    const data = await response.text();
    const parsedData = d3.csvParse(data, d => ({
      most_serious_bias_type: d['most_serious_bias_type'],
      value: parseFloat(d[selectedTerm])
    }));
    return parsedData;
  }

  function updateChart(data) {
  const margin = { top: 30, right: 30, bottom: 30, left: 60 };
  const width = innerWidth*0.4;
  const height = innerHeight-200;

  // Select the SVG, create it if it doesn't exist
  let svg = d3.select("#bar-chart").select("svg");
  if (currentSection > 0){ 
    const x = d3.scaleBand()
      .range([0, width])
      .padding(0.1)
      .domain(data.map(d => d.most_serious_bias));

    const y = d3.scaleLinear()
      .range([height, 0])
      .domain([0, 450]);

    // Update axes
    svg.select(".x-axis").transition().duration(500).call(d3.axisBottom(x));
    svg.select(".y-axis").transition().duration(500).call(d3.axisLeft(y));

    // Tooltip
    const tooltip = d3.select("body")
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("visibility", "hidden")
        .style("background-color", "white")
        .style("border", "1px solid #ccc")
        .style("padding", "10px")
        .style("border-radius", "4px")
        .on("mouseover", (event, d) => {
          tooltip.style("visibility", "hidden")
        });

    const chartContain = svg.select('.chart-contain')

    // Bind data to bars
    const bars = chartContain.selectAll(".bar")
      .data(data, d => d.most_serious_bias);


    // Update existing bars
    bars.transition()
      .duration(500)
      .attr("x", d => x(d.most_serious_bias))
      .attr("width", x.bandwidth())
      .attr("y", d => y(d.value))
      .attr("height", d => height - y(d.value));

    // Enter new bars
    bars.enter()
      .append("rect")
      .attr("class", "bar")
      .attr("x", d => x(d.most_serious_bias))
      .attr("width", x.bandwidth())
      .attr("y", y(0)) // Start from bottom
      .attr("height", 0) // Initially no height
      .attr("fill", "#aa334f")
      .on("mouseover", (event, d) => {
          tooltip.style("visibility", "visible")
                .text(`${d.most_serious_bias}: ${d.value}`);
        })
        .on("mousemove", event => {
          tooltip.style("top", (event.pageY - 10) + "px")
                .style("left", (event.pageX + 10) + "px");
        })
        .on("mouseout", () => {
          tooltip.style("visibility", "hidden");
        })
      .merge(bars) // Combine enter and update selections
      .transition()
      .duration(500)
      .attr("y", d => y(d.value))
      .attr("height", d => height - y(d.value));

    // Exit and remove old bars
    bars.exit()
      .transition()
      .duration(500)
      .attr("y", y(0))
      .attr("height", 0)
      .remove();
    } else {
      const bars = svg.selectAll(".bar")
      bars.exit()
       .transition()
       .on("end", function() {
        if (svg.selectAll(".bar").empty()) {
            tooltip.style("visibility", "hidden");
        }
       })
       .remove();
      svg.remove();
    }
}

function drawBar(data) {
  const margin = { top: 30, right: 30, bottom: 30, left: 60 };
  const width = innerWidth * 0.4;
  const height = innerHeight - 200;

  // Define a color scale
  const colorScale = d3.scaleOrdinal(d3.schemeTableau10);

  // Select the SVG, create it if it doesn't exist
  let svg = d3.select("#bar-chart").select("svg");
  if (currentSection >= 0) {
    if (svg.empty()) {
      svg = d3.select("#bar-chart")
              .append("svg")
              .attr("width", width + margin.left + margin.right)
              .attr("height", height + margin.top + margin.bottom)
              .append("g")
              .attr("class", "chart-contain")
              .attr("transform", `translate(${margin.left},${margin.top})`);

      // Append the g elements for axes only once
      svg.append("g").attr("class", "x-axis")
        .attr("transform", `translate(0,${height})`)
        .selectAll("text")
          .style("text-anchor", "end")
          .attr("dx", "-.8em")
          .attr("dy", ".15em");

      // d3.select('.x-axis').selectAll(this.childNodes).attr("opacity", "0.5");
      svg.append("g").attr("class", "y-axis");
      
      svg.append("text")
        .attr("class", "x-axis-label")
        .attr("text-anchor", "middle")
        .attr("x", width / 2)
        .attr("y", height + margin.bottom + 10)
        .text("Bias Type");

      svg.append("text")
        .attr("class", "y-axis-label")
        .attr("text-anchor", "middle")
        .attr("transform", "rotate(-90)")
        .attr("x", -height / 2)
        .attr("y", -margin.left + 20)
        .text("Number of Hate Crime Incidents");

      svg.append("text")
        .attr("class", "chart-title")
        .attr("text-anchor", "middle")
        .attr("x", width / 2)
        .attr("y", -margin.top / 2)
        .attr("font-size", "20px")
        .attr("font-weight", "bold")
        .attr('width', '20%')
        .attr('overflow-wrap', 'break-word')
        .text("Number of Hate Crime Incidents In San Francisco");
    }

    const x = d3.scaleBand()
      .range([0, width])
      .padding(0.1)
      .domain(data.map(d => d.most_serious_bias_type));

    const y = d3.scaleLinear()
      .range([height, 0])
      .domain([0, 450]);

    // Update axes
    svg.select(".x-axis").transition().duration(500).call(d3.axisBottom(x));
    svg.select(".y-axis").transition().duration(500).call(d3.axisLeft(y));

    // Tooltip
    const tooltip = d3.select("body")
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("visibility", "hidden")
        .style("background-color", "white")
        .style("border", "1px solid #ccc")
        .style("padding", "10px")
        .style("border-radius", "4px");

    const chartContain = svg.select('.chart-contain');

    // Bind data to bars
    const bars = chartContain.selectAll(".bar")
      .data(data, d => d.most_serious_bias_type);

    // Enter new bars
    bars.enter()
      .append("rect")
      .attr("class", "bar")
      .attr("x", d => x(d.most_serious_bias_type))
      .attr("width", x.bandwidth())
      .attr("y", y(0))
      .attr("height", 0)
      .attr("fill", d => colorScale(d.most_serious_bias_type))
      .on("mouseover", (event, d) => {
        tooltip.style("visibility", "visible")
               .text(`${d.most_serious_bias_type}: ${d.value}`);
      })
      .on("mousemove", event => {
        tooltip.style("top", (event.pageY - 10) + "px")
               .style("left", (event.pageX + 10) + "px");
      })
      .on("mouseout", () => {
        tooltip.style("visibility", "hidden");
      })
      .merge(bars) // Combine enter and update selections
      .transition()
      .duration(500)
      .attr("y", d => y(d.value))
      .attr("height", d => height - y(d.value));

    // Exit and remove old bars
    bars.exit()
      .transition()
      .duration(500)
      .attr("y", y(0))
      .attr("height", 0)
      .remove();
    } else {
      const bars = svg.selectAll(".bar")
      bars.exit()
       .transition()
       .on("end", function() {
        if (svg.selectAll(".bar").empty()) {
            tooltip.style("visibility", "hidden");
        }
       })
       .remove();
      svg.remove();
    }
}

const drawLineGraph = (svg, filteredData, width, height, margin, bias) => {
  // Selector
  const biasTypes = ["Total", "Disability", "Gender", "Gender Nonconforming", "Race", "Religion", "Sexual Orientation"]

  const colorScale = d3.scaleOrdinal()
    .domain(biasTypes)
    .range(d3.schemeCategory10);

  const parseDate = d3.timeParse("%Y");

  filteredData.forEach(d => {
    d.occurence_year = parseDate(d.occurence_year);
    d.total_number_of_individual_victims = +d.total_number_of_individual_victims;
  });

  // Set up scales
  const x = d3.scaleTime()
    .domain(d3.extent(filteredData, d => d.occurence_year))
    .range([0, width]);

  const y = d3.scaleLinear()
    .domain([0, d3.max(filteredData, d => d.total_number_of_individual_victims)])
    .nice()
    .range([height, 0]);

  const line = d3.line()
    .x(d => x(d.occurence_year))
    .y(d => y(d.total_number_of_individual_victims));

  let path = svg.append("path")
    .datum(filteredData)
    .attr("fill", "none")
    .attr("stroke", colorScale(bias))
    .attr("stroke-width", 2)
    .attr("d", line);

  path.attr("stroke-dasharray", function() {
    const length = this.getTotalLength();
    return length + "," + length;
  })
  .attr("stroke-dashoffset", function() {
    return this.getTotalLength();
  })
  .transition()
  .duration(3000)
  .ease(d3.easeLinear)
  .attr("stroke-dashoffset", 0);


  svg.append("g")
    .attr("transform", `translate(0, ${height})`)
    .call(d3.axisBottom(x));

  svg.append("g")
    .call(d3.axisLeft(y));

  svg.append("text")
    .attr("class", "x-axis-label")
    .attr("text-anchor", "middle")
    .attr("x", width/2)
    .attr("y", height + margin.bottom + 10)
    .text("Year")

  svg.append("text")
    .attr("class", "y-axis-label")
    .attr("text-anchor", "middle")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", -margin.left + 20)
    .text(`Number of ${bias} Hate Crime Victims`);

  svg.append("text")
    .attr("class", "chart-title")
    .attr("text-anchor", "middle")
    .attr("x", width / 2)
    .attr("y", -margin.top / 2)
    .attr("font-size", "20px")
    .attr("font-weight", "bold")
    .text(`${bias} Hate Crime Victims In San Francisco`);

  const tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background-color", "white")
    .style("border", "1px solid #ccc")
    .style("padding", "10px")
    .style("border-radius", "4px");

  // Add mouseover event handlers for displaying tooltip
  svg.selectAll("circle")
    .data(filteredData)
    .enter()
    .append("circle")
    .attr("cx", d => x(d.occurence_year))
    .attr("cy", d => y(d.total_number_of_individual_victims))
    .attr("r", 5)
    .attr("fill", "#144463")
    .on("mouseover", (event, d) => {
      tooltip.style("visibility", "visible")
        .html(`<strong>Year:</strong> ${d.occurence_year.getFullYear()}<br><strong>Total Victims:</strong> ${d.total_number_of_individual_victims}`);
    })
    .on("mousemove", event => {
      tooltip.style("top", (event.pageY - 10) + "px")
        .style("left", (event.pageX + 10) + "px");
    })
    .on("mouseout", () => {
      tooltip.style("visibility", "hidden");
    });

}

const numOverTime = () => {
  // if (currentSection >= -1){
    const lineSVG = d3.select("#line-chart").select("svg")
    if (!lineSVG.empty()) {
      lineSVG.remove()
    }
    d3.csv(`${base}/victims_over_time_by_bias.csv`).then(data => {
      const margin = { top: 30, right: 30, bottom: 30, left: 60 };
      const width = innerWidth*0.4;
      const height = innerHeight-200;

      const svg = d3.select("#line-chart").append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom + 20)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      const selectedBias = document.getElementById('bias-selector').value;
      
      const filteredData = data.filter(d => d.most_serious_bias_type == selectedBias)

      drawLineGraph(svg, filteredData, width, height, margin, selectedBias);
    });
  // }
}

</script>

<svelte:window bind:innerWidth />

<main>
  <div class="left">
    <img id="cover-img" src="https://images.unsplash.com/photo-1591259622709-bdb033b4be2b?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" alt="protest">
    <div id="line-chart"></div>
    <div id="bias-select">
      <label for="bias-selector">Select Bias Type:</label>
      <select id="bias-selector">
        <option value="Total">Total</option>
        <option value="Disability">Disability</option>
        <option value="Gender">Gender</option>
        <option value="Gender Nonconforming">Gender Nonconforming</option>
        <option value="Race">Race</option>
        <option value="Religion">Religion</option>
        <option value="Sexual Orientation">Sexual Orientation</option>
      </select>
    </div>
    <div id="bar-chart" style="position: sticky; top: 10px;"></div>
  </div>
  <div class="right">
    <h1><b>Examining the Link</b>: The Rise of Hate Crimes And Social Justice Movements In San Francisco</h1>
    <p id="names">by Angela Hu, Colin Wang & Justin Lu</p>
    {#each terms as term, i}
      <section class="blurbs">
        <h2>{term}</h2>
        <img src={images[i]}/>
        <p>{blurbs[i]}</p>
      </section>
    {/each}
    <section class="blurbs">
      <h2>Sub-Categories of Bias</h2>
      <select id="bias-type-select" name="bias">
        <option value="Disability">Disability</option>
        <option value="Gender">Gender</option>
        <option value="Gender Nonconforming">Gender Nonconforming</option>
        <option value="Race" selected="selected">Race</option>
        <option value="Religion">Religion</option>
        <option value="Sexual Orientation">Sexual Orientation</option>
      </select>
      <p>Within each bias type there are various types of motivaitons for each hate crime. We can see that within broad categories that certain communities are at greater risk of being victims</p>
    </section>
    <section>
      <h2 id="conc">Conclusion</h2>
      <p id="conc-text">Looking at how rates change between terms, one noteworthy data point is how although race-based hate crimes were running rampant during Bush’s first term with the war on terrorism, we can see them taking a backseat to sexuality-based violence in Bush’s second term, potentially correlated to the LGBTQ rights legislation worked on at the time.
        Overall, we hope that viewers learn more about the varying social movements and how hate crimes and their types can change based on the political landscape, all through our time-based scrollytelling.</p>
    </section>
  </div>
</main>






<style>
  @import '../../static/css/styles.css';

  .bar {
    fill: var(--primary);
  }

  svg {
    background-color: #f5f5f5;
    border: 1px solid #ccc;
  }

  .blurbs {
    display: flex;
    flex-direction: column;
    justify-content: center;
    min-height: 100vh; 
    padding: 20px;
  }
  
</style>
