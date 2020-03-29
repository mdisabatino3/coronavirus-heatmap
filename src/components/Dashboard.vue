<template>
  <div class="custom-card header-card card">
    <div class="card-body pt-0">
      <svg width="975" height="610"/>
    </div>
  </div>
</template>

<script>

export default {
  props: [],
  components: {},
  data() {
    return {

    }
  },
  mounted: async function() {
    var svg = d3.select("svg").style("margin-top","20px");
    var path = d3.geoPath();
    var tooltipHeight = 100;
    var tooltipWidth = 200;

    var zoom = d3.zoom().on("zoom", zoomed);

    var g = svg.append("g");

    var div;

    var stateIds = [];

    var colorScale = d3.scaleLinear()
      .interpolate(d3.interpolateRgb)
      .range([d3.rgb("#FFFFFF"), d3.rgb('#710019')]);


    svg.call(zoom).call(zoom.transform, d3.zoomIdentity.translate(-3600,-800).scale(5)); // delete this line to disable free zooming
    // .call(zoom.event); // not in d3 v4

    var tooltip = d3.select(".card-body").append("div") 
      .attr("class", "tooltip")       
      .style("opacity", 0)
      .style("height", tooltipHeight)
      .style("width", tooltipWidth);
    // var usStates =d3.json("https://d3js.org/us-10m.v1.json").then(function(us) {
    var usStates =d3.json("https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json").then(function(us) {

      us.objects.states.geometries.forEach(geometry => {
        stateIds.push({
          id: geometry.id,
          name: geometry.properties.name 
        })
      })

      stateIds.sort((a,b) => (a.id > b.id) ? 1 : -1);

      d3.csv("https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties.csv").then(function(covidData) {
        var extent = d3.extent(covidData, function(d) {return parseInt(d.cases);})
        extent[1] = extent[1]/25
        extent[0] = 0;
        colorScale.domain(extent);
        us.objects.counties.geometries.forEach(function(d) {
          d.properties.id = d.id;
          d.properties.stateId = d.id.toString().slice(0,2);
          let state = stateIds.find(el => el.id === d.properties.stateId)
          d.properties.stateName = state.name;
          let exceptional = getCaseDataForExceptions(d.properties.id);
          let caseData;
          let color;
          if (exceptional !== null) {
            caseData = covidData.filter(el => el.county === exceptional[0]).sort(function(a,b) {
              a = new Date(a.date);
              b = new Date(b.date);
              return a < b ? 1 : -1;
            });
            if (caseData.length !== 0) {
              caseData = caseData[0];
            } else {
              caseData = null;
            }
            color = colorScale(caseData !== null ? caseData.cases : 0);
          } else {
            caseData = covidData.filter(el => el.fips === d.properties.id).sort(function(a,b) {
              a = new Date(a.date);
              b = new Date(b.date);
              return a < b ? 1 : -1;
            });
            if (caseData.length !== 0) {
              caseData = caseData[0];
            } else {
              caseData = null;
            }
            color = colorScale(caseData !== null ? caseData.cases : 0);
          }
          d.properties.caseData = caseData;
          d.properties.color = color;
        })

        setColors();

      });

      g.selectAll("path")
        .data(topojson.feature(us, us.objects.counties).features)
        .enter().append("path")
          .attr("class","counties")
          .attr("d", path)
          .attr("id", "county")
          .style("stroke","#D2D2D2")
          .style("opacity",0.7)
          .on("click", clicked)
          .on("mouseover", mouseover)
          .on("mousemove",mousemove)
          .on("mouseout",mouseout);

      g.append("path")
        .attr("class", "state-borders")
        .attr("d", path(topojson.mesh(us, us.objects.states, function(a, b) { return a !== b; })))
        .style("stroke", "black");

      var focus;
      function mouseover() {
        focus = d3.select(this).style("opacity",1).style("stroke","#aa0026");
        let cardBody = d3.select(".card-body").node().getBoundingClientRect();
        let offsetTop = cardBody.y;
        let offsetLeft = cardBody.x
        let dataProperties = d3.select(this).data()[0].properties;
        let countyId = dataProperties.id;
        let countyName = dataProperties.name;
        let stateName = dataProperties.stateName;
        let numCases = dataProperties.caseData !== null ? dataProperties.caseData.cases : "no cases reported";
        let numDeaths = dataProperties.caseData !== null ? dataProperties.caseData.deaths : "no deaths reported";
        let exceptional = getCaseDataForExceptions(countyId);
        if (exceptional === null) {
          tooltip
              .style("opacity", 0.9);    
              tooltip.html("<p><strong>" + dataProperties.name + ", " + dataProperties.stateName + "</strong><p>" +
              "<p>Cases: " + numCases + "</p>" +
              "<p>Deaths: " + numDeaths)  
              .style("left", (d3.event.pageX - offsetLeft - tooltipWidth*1.1) + "px")   
              .style("top", (d3.event.pageY - offsetTop - tooltipHeight*1.5) + "px");
        } else {
          tooltip
              .style("opacity", 0.9);    
              tooltip.html("<p><strong>" + dataProperties.name + ", " + dataProperties.stateName + "</strong><p>" +
              "<p>Cases for " + exceptional[0] + ": " + numCases + "</p>" +
              "<p>Deaths for " + exceptional[0] + ": " + numDeaths)  
              .style("left", (d3.event.pageX - offsetLeft - tooltipWidth) + "px")   
              .style("top", (d3.event.pageY - offsetTop - tooltipHeight*1.3) + "px");
        }
      }

      function mousemove() {
        //
      }

      function mouseout() {
        focus.style("opacity",0.7).style("stroke","#D2D2D2");
        tooltip   
            .style("opacity", 0) 
      }


    });

    function setColors() {
      console.log("called set colors");
      d3.selectAll(".counties")
        .style("fill", function(d) {
          return d.properties.color;
        })
    }

    
    function clicked() {
    }

    function zoomed() {
      g.style("stroke-width", 1.5 / d3.event.transform.k + "px");
      // g.attr("transform", "translate(" + d3.event.translate + ")scale(" + d3.event.scale + ")"); // not in d3 v4
      g.attr("transform", d3.event.transform); // updated for d3 v4

      console.log(" exiting paths " + JSON.stringify(g.selectAll("path").exit()));
    }

    // Exceptions for New York, Kings, Queens, Bronx, Richmond -> New York City
    // Exceptions for Cass, Clay, Jackson, and Platte Kansas -> Kansas City
    function getCaseDataForExceptions(countyId) {
      let newYorkCityZips = ["36081", "36047", "36005", "36085","36061"];
      let kansasCityZips = ["29037", "29047", "29095", "29165"];
      if (newYorkCityZips.includes(countyId)) {
        return ["New York City","New York"];
      } else if (kansasCityZips.includes(countyId)) {
        return ["Kansas City", "Missouri"];
      } else {
        return null;
      }
    }
  }
    
}
</script>

<style lang="scss">
  #county {
    fill: #D2D2D2;
    &:hover {
      fill: red;
      opacity: 0.5;
    }
  }

  .state-borders {
    fill: none
  }

  .county-borders {
    fill: none;
    stroke: #121212;
    stroke-width: 0.5px;
    stroke-linejoin: round;
    stroke-linecap: round;
    pointer-events: none;
  }

  text {
    font-weight: 500;
    font-size: 18px;
    color: #081f2c;
  }

  div.tooltip { 
    position: absolute;     
    text-align: left;     
    width: 200px;          
    height: 120px;         
    padding: 12px;       
    font: 12px sans-serif;
    color: #EBEBEB;   
    background: #3b4850; 
    border: 0px;        
    pointer-events: none;
    border-radius: 10%;     
  }
</style>