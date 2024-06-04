<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { base } from '$app/paths';

  let currentSection = 0;
  let innerWidth;

  const terms = [
    "2001-2004: George W. Bush's 1st Term",
    "2005-2008: George W. Bush's 2nd Term",
    "2009-2012: Barack Obama's 1st Term",
    "2013-2016: Barack Obama's 2nd Term",
    "2017-2020: Donald Trump's Term",
    "2021-2024: Biden's Term"
  ];

  const blurbs = [
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

  const images = ['https://images.wsj.net/im-394615?width=1920',
  'https://static01.nyt.com/images/2009/02/08/business/08stream_600.jpg?quality=75&auto=webp',
  'https://www.matthewshepard.org/wp-content/uploads/2020/04/Obama-passing-prevention-act-1.jpg',
  'https://assets.apnews.com/05/85/8194b8d0be0147eb15649784e23a/04c53adcfa5341a29da266a6aba12d66',
  'https://static.politico.com/70/8e/53affa2c466695e48a9063551bda/20210112-trump-wall-ap-773.jpg',
  'https://media-cldnry.s-nbcnews.com/image/upload/t_fit-1240w,f_auto,q_auto:best/rockcms/2021-04/210413-Bellevue-stop-aapi-hate-rally-ac-424p-4e6a6a.jpg'
  ]

  onMount(() => {
    window.addEventListener('scroll', handleScroll);
    loadDataAndChart(terms[0]); // Load initial data
  });

  function handleScroll() {
    const sectionHeight = window.innerHeight;
    const newSection = Math.min(terms.length - 1, Math.floor(window.scrollY / sectionHeight) - 1);
    if (newSection !== currentSection) {
      currentSection = newSection;
      loadDataAndChart(terms[currentSection]);
    }
  }

  async function loadDataAndChart(selectedTerm) {
    const data = await load_data(selectedTerm);
    updateChart(data);
  }

  async function load_data(selectedTerm) {
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
  if (currentSection !== -1){ 
    if (svg.empty()) {
      svg = d3.select("#bar-chart")
              .append("svg")
              .attr("width", width + margin.left + margin.right)
              .attr("height", height + margin.top + margin.bottom + 20)
              .append("g")
              .attr("transform", `translate(${margin.left},${margin.top})`);


    // var xAxis = d3.svg.axis()
      // .scale(x);

      // Append the g elements for axes only once
      svg.append("g").attr("class", "x-axis")
        .attr("transform", `translate(0,${height}) `)
        .selectAll("text")
          .attr("transform", "rotate(-65)")


        //  .attr("transform", "")
        //  .width('10px')
        //   .selectAll("text")  
            .style("text-anchor", "end")
            .attr("dx", "-.8em")
            .attr("dy", ".15em")
        //     .attr("transform", "rotate(-65)" );
        .style("text-anchor", "end");

      svg.append("g").attr("class", "y-axis")
      
      svg.append("text")
        .attr("class", "x-axis-label")
        .attr("text-anchor", "middle")
        .attr("x", width/2)
        .attr("y", height + margin.bottom + 10)
        .text("Bias Type")

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
        .text("Number of Hate Crime Incidents In San Francisco");
    }

    const x = d3.scaleBand()
      .range([0, width])
      .padding(0.1)
      .domain(data.map(d => d.most_serious_bias_type));

    const y = d3.scaleLinear()
      .range([height, 0])
      .domain([0, 400]);

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

    // Bind data to bars
    const bars = svg.selectAll(".bar")
      .data(data, d => d.most_serious_bias_type);

    // Update existing bars
    bars.transition()
      .duration(500)
      .attr("x", d => x(d.most_serious_bias_type))
      .attr("width", x.bandwidth())
      .attr("y", d => y(d.value))
      .attr("height", d => height - y(d.value));

    // Enter new bars
    bars.enter()
      .append("rect")
      .attr("class", "bar")
      .attr("x", d => x(d.most_serious_bias_type))
      .attr("width", x.bandwidth())
      .attr("y", y(0)) // Start from bottom
      .attr("height", 0) // Initially no height
      .attr("fill", "steelblue")
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
      // .attr("y", y(0))
      // .attr("height", 0)
       .on("end", function() {
        if (svg.selectAll(".bar").empty()) {
            tooltip.style("visibility", "hidden");
        }
       })
       .remove();
      //bars.remove();
      svg.remove();
    }
}

</script>

<svelte:window bind:innerWidth />

<main>
  <div class="left">
    <div id="bar-chart" style="position: sticky; top: 10px;"></div>
  </div>
  <div class="right">
    <h1>Examining the Link: The Rise of Hate Crimes And Social Justice Movements In San Francisco</h1>
    {#each terms as term, i}
      <section class="blurbs">
        <h2>{term}</h2>
        <img src={images[i]}/>
        <p>{blurbs[i]}</p>
      </section>
    {/each}
  </div>
</main>






<style>
  @import '../../static/css/styles.css';

  .bar {
    fill: steelblue;
  }

  svg {
    background-color: #f4f4f4;
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
