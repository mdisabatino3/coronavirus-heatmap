<template>
  <div class="container-fluid" id="app">
    <div class="card-body pt-0">
      <div class="row">
        <div id="sidebar" class="col-md-3 col-sm-12 col-xs-12 sidebar shadow">
          <div id="selection">
            <select v-model="selected" @change="selectCounty($event)">
              <option disabled value="">Please select one</option>
              <option>Chester County</option>
              <option>Delaware County</option>
              <option>Montgomery County</option>
            </select>
          </div>
          <div id="scrollable">
            <h2 id="info"><strong>{{ selected }}</strong></h2>
            <ul id="countyList">
              <li v-for="township in data" :key="township.ID" id="countyListItem">
                <h3>{{township.ID}}</h3>
                <p id="townshipDetail">Cases: {{township.Value}}</p>
                <p id="townshipDetail">Deaths: {{township.C}}</p>
              </li>
            </ul>
          </div>
        </div>
        <!-- <div class="col-md-12 col-sm-12 col-xs-12" id="banner">Vitable Coronavirus Heatmap</div> -->
        <div class="col-md-8 col-sm-12 col-xs-12">
          <div id="mapwrapper" class="shadow">
            <div id="banner" class="row"><h1><strong>Vitable Coronavirus Heatmap</strong></h1></div>
            <div id="maprow" class="row">
              <div id="map">
                <div class="loader"></div>
                <svg width="960" height="600"/>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  props: [],
  components: {},
  data() {
    return {
      selected: 'Delaware County',
      countyData: {},
      data: [],
    }
  },
  methods: {
    selectCounty(event) {
      console.log("setting county " + event.target.value)
      let countyName = event.target.value;
      this.data = this.countyData[countyName];
      console.log("data updated " + this.data);
    },
    selectCountyOnMount(countyName) {
      console.log("setting county " + countyName);
      console.log("county data " + JSON.stringify(this.countyData));
      this.data = this.countyData[countyName];
      console.log("data updated " + JSON.stringify(this.data));
    },
    setCountyData(countyName, data) {
      this.countyData[countyName] = data;
    }
  },
  mounted: async function() {
    var chesterCountyCsv = "data:application/octet-stream;charset=utf-8,ID%2CValue%2CC%0AWest%20Grove%20Borough%2C0%2C0%0ANew%20London%20Township%2C0%2C0%0AHoney%20Brook%20Borough%2C0%2C0%0AEast%20Fallowfield%20Township%2C1%2C0%0ACaln%20Township%2C2%2C0%0AValley%20Township%2C0%2C0%0AElk%20Township%2C0%2C0%0AOxford%20Borough%2C1%2C0%0AWillistown%20Township%2C3%2C0%0AAvondale%20Borough%2C0%2C0%0AWest%20Fallowfield%20Township%2C0%2C0%0AAtglen%20Borough%2C0%2C0%0AFranklin%20Township%2C0%2C0%0APhoenixville%20Borough%2C0%2C0%0AParkesburg%20Borough%2C0%2C0%0ASouth%20Coventry%20Township%2C0%2C0%0AWest%20Nottingham%20Township%2C0%2C0%0AWest%20Marlborough%20Township%2C0%2C0%0ASpring%20City%20Borough%2C1%2C0%0ALondon%20Britain%20Township%2C1%2C0%0AEast%20Nantmeal%20Township%2C0%2C0%0AWest%20Caln%20Township%2C1%2C0%0AEast%20Caln%20Township%2C0%2C0%0ASchuylkill%20Township%2C0%2C0%0ACity%20of%20Coatesville%2C0%2C0%0AThornbury%20Township%2C0%2C0%0AEast%20Goshen%20Township%2C5%2C0%0AWest%20Pikeland%20Township%2C3%2C0%0AKennett%20Township%2C0%2C0%0AEast%20Marlborough%20Township%2C2%2C0%0AWest%20Chester%20Borough%2C7%2C0%0AEast%20Brandywine%20Township%2C1%2C0%0ADowningtown%20Borough%2C1%2C0%0AWest%20Brandywine%20Township%2C0%2C0%0AEast%20Bradford%20Township%2C2%2C0%0AWallace%20Township%2C0%2C0%0AEast%20Whiteland%20Township%2C8%2C0%0AEast%20Coventry%20Township%2C1%2C0%0ABirmingham%20Township%2C2%2C0%0ANew%20Garden%20Township%2C1%2C0%0ANorth%20Coventry%20Township%2C1%2C0%0ANewlin%20Township%2C0%2C0%0AWest%20Goshen%20Township%2C3%2C0%0APenn%20Township%2C0%2C0%0AWest%20Whiteland%20Township%2C4%2C0%0APennsbury%20Township%2C2%2C0%0AEast%20Pikeland%20Township%2C3%2C0%0AEast%20Vincent%20Township%2C3%2C0%0AUpper%20Oxford%20Township%2C1%2C0%0AWest%20Vincent%20Township%2C1%2C0%0AHoney%20Brook%20Township%2C2%2C0%0AUpper%20Uwchlan%20Township%2C3%2C0%0AWest%20Bradford%20Township%2C1%2C0%0AUwchlan%20Township%2C8%2C0%0ALondonderry%20Township%2C0%2C0%0ASouth%20Coatesville%20Borough%2C0%2C0%0ALower%20Oxford%20Township%2C0%2C0%0AEasttown%20Township%2C9%2C0%0AWesttown%20Township%2C0%2C0%0ALondon%20Grove%20Township%2C0%2C0%0AKennett%20Square%20Borough%2C1%2C0%0AEast%20Nottingham%20Township%2C1%2C0%0AMalvern%20Borough%2C1%2C0%0ACharlestown%20Township%2C2%2C0%0ATredyffrin%20Township%2C1%2C0%0AModena%20Borough%2C0%2C0%0AWest%20Nantmeal%20Township%2C0%2C0%0AWarwick%20Township%2C0%2C0%0AElverson%20Borough%2C0%2C0%0APocopson%20Township%2C0%2C0%0AHighland%20Township%2C0%2C0%0ASadsbury%20Township%2C0%2C0%0AWest%20Sadsbury%20Township%2C0%2C0";
    var delawareCountyCsv = "data:application/octet-stream;charset=utf-8,ID%2CValue%2CC%0AYeadon%20Borough%2C2%2C0%0AAldan%20Borough%2C1%2C0%0ATinicum%20Township%2C0%2C0%0ARose%20Valley%20Borough%2C2%2C0%0ALower%20Chichester%20Township%2C0%2C0%0ALansdowne%20Borough%2C1%2C0%0ARidley%20Township%2C6%2C0%0AProspect%20Park%20Borough%2C1%2C0%0AHaverford%20Township%2C21%2C0%0AChadds%20Ford%20Township%2C1%2C0%0ARidley%20Park%20Borough%2C4%2C1%0ASwarthmore%20Borough%2C4%2C0%0AUpland%20Borough%2C0%2C0%0AEdgmont%20Township%2C0%2C0%0AMarcus%20Hook%20Borough%2C0%2C0%0AParkside%20Borough%2C0%2C0%0AMillbourne%20Borough%2C1%2C0%0AUpper%20Chichester%20Township%2C5%2C0%0ACollingdale%20Borough%2C1%2C0%0AColwyn%20Borough%2C0%2C0%0AGlenolden%20Borough%2C0%2C0%0AChester%20Heights%20Borough%2C0%2C0%0AUpper%20Providence%20Township%2C4%2C0%0AMarple%20Township%2C11%2C1%0ANorwood%20Borough%2C1%2C0%0AClifton%20Heights%20Borough%2C3%2C0%0AAston%20Township%2C5%2C0%0AChester%20Township%2C1%2C0%0ASpringfield%20Township%2C7%2C0%0ARutledge%20Borough%2C0%2C0%0ADarby%20Township%2C0%2C0%0AConcord%20Township%2C4%2C0%0AThornbury%20Township%2C5%2C0%0AMorton%20Borough%2C3%2C0%0AMiddletown%20Township%2C10%2C1%0ADarby%20Borough%2C1%2C0%0AEddystone%20Borough%2C1%2C0%0ABrookhaven%20Borough%2C1%2C0%0ARadnor%20Township%2C21%2C0%0ATrainer%20Borough%2C1%2C0%0ANewtown%20Township%2C11%2C0%0AEast%20Lansdowne%20Borough%2C2%2C0%0AFolcroft%20Borough%2C0%2C0%0AUpper%20Darby%20Township%2C27%2C0%0ASharon%20Hill%20Borough%2C12%2C0%0ABethel%20Township%2C2%2C0%0AMedia%20Borough%2C2%2C0%0AChester%20City%2C3%2C0%0ANether%20Providence%20Township%2C9%2C0";
    var montgomeryCountyCsv = "data:application/octet-stream;charset=utf-8,ID%2CValue%2CC%0AUpper%20Merion%20Township%2C13%2C0%0ACheltenham%20Township%2C28%2C1%0ABridgeport%20Borough%2C0%2C0%0AJenkintown%20Borough%2C2%2C0%0AConshohocken%20Borough%2C5%2C0%0ARockledge%20Borough%2C0%2C0%0AWest%20Conshohocken%20Borough%2C0%2C0%0ALower%20Merion%20Township%2C68%2C0%0AUpper%20Hanover%20Township%2C1%2C0%0AEast%20Greenville%20Borough%2C0%2C0%0APennsburg%20Borough%2C0%2C0%0AWest%20Pottsgrove%20Township%2C1%2C0%0AMontgomery%20Township%2C6%2C0%0ATowamencin%20Township%2C4%2C0%0APottstown%20Borough%2C5%2C0%0ASchwenksville%20Borough%2C2%2C0%0APerkiomen%20Township%2C4%2C0%0ALansdale%20Borough%2C4%2C0%0ASkippack%20Township%2C6%2C0%0AUpper%20Gwynedd%20Township%2C4%2C0%0AHorsham%20Township%2C7%2C0%0AWorcester%20Township%2C10%2C0%0ANarberth%20Borough%2C2%2C0%0ALimerick%20Township%2C8%2C0%0ARoyersford%20Borough%2C1%2C0%0AWest%20Norriton%20Township%2C5%2C0%0AAbington%20Township%2C31%2C2%0AUpper%20Providence%20Township%2C11%2C0%0ANorth%20Wales%20Borough%2C0%2C0%0ALower%20Gwynedd%20Township%2C9%2C0%0ATrappe%20Borough%2C1%2C0%0ACollegeville%20Borough%2C3%2C0%0ALower%20Providence%20Township%2C27%2C0%0AWhitpain%20Township%2C10%2C0%0AUpper%20Moreland%20Township%2C4%2C0%0AHatboro%20Borough%2C1%2C0%0AUpper%20Dublin%20Township%2C14%2C0%0AEast%20Norriton%20Township%2C6%2C0%0AAmbler%20Borough%2C5%2C0%0ALower%20Moreland%20Township%2C10%2C0%0ABryn%20Athyn%20Borough%2C0%2C0%0ADouglass%20Township%2C1%2C0%0AMarlborough%20Township%2C0%2C0%0ARed%20Hill%20Borough%2C0%2C0%0ANew%20Hanover%20Township%2C2%2C0%0ASalford%20Township%2C1%2C0%0AUpper%20Frederick%20Township%2C3%2C0%0AGreen%20Lane%20Borough%2C0%2C0%0AFranconia%20Township%2C0%2C0%0AUpper%20Salford%20Township%2C0%2C0%0ATelford%20Borough%2C0%2C0%0ASouderton%20Borough%2C3%2C0%0ALower%20Frederick%20Township%2C0%2C0%0AHatfield%20Township%2C2%2C0%0AUpper%20Pottsgrove%20Township%2C1%2C0%0ALower%20Salford%20Township%2C4%2C0%0AHatfield%20Borough%2C0%2C0%0ALower%20Pottsgrove%20Township%2C2%2C0%0AWhitemarsh%20Township%2C9%2C1%0APlymouth%20Township%2C9%2C0%0ANorristown%20Borough%2C4%2C0%0ASpringfield%20Township%2C12%2C0";
    var svg = d3.select("svg").style("margin-top","0px").style("display","none").style("background-color","#D2D2D2");
    var path = d3.geoPath();
    var tooltipHeight = 60;
    var tooltipWidth = 200;

    var zoom = d3.zoom().on("zoom", zoomed);

    var g = svg.append("g");

    var div;

    var component = this;

    var stateIds = [];
    d3.csv(chesterCountyCsv).then(function(d) {
      console.log(d);
      component.setCountyData("Chester County", d);
    })
    d3.csv(delawareCountyCsv).then(function(d) {
      console.log(d);
      component.setCountyData("Delaware County", d);
      component.selectCountyOnMount("Delaware County");
    })
    d3.csv(montgomeryCountyCsv).then(function(d) {
      console.log(d);
      component.setCountyData("Montgomery County", d);
    });

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

      g.append("path")
        .attr("d",path(topojson.merge(us, us.objects.states.geometries.filter(function(d) { return d.id; }))))
        .attr("class", "us-border");

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
              .style("left", (d3.event.pageX - offsetLeft - tooltipWidth*.8) + "px")   
              .style("top", (d3.event.pageY - offsetTop - tooltipHeight*1.5) + "px");
        } else {
          tooltip
              .style("opacity", 0.9);    
              tooltip.html("<p><strong>" + dataProperties.name + ", " + dataProperties.stateName + "</strong><p>" +
              "<p>Cases for " + exceptional[0] + ": " + numCases + "</p>" +
              "<p>Deaths for " + exceptional[0] + ": " + numDeaths)  
              .style("left", (d3.event.pageX - offsetLeft - tooltipWidth*0.8) + "px")   
              .style("top", (d3.event.pageY - offsetTop - tooltipHeight*1.5) + "px");
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
      svg.style("display", "block");
      d3.select(".loader").style("display","none");
    }

    
    function clicked() {
    }

    function zoomed() {
      g.style("stroke-width", 1.5 / d3.event.transform.k + "px");
      // g.attr("transform", "translate(" + d3.event.translate + ")scale(" + d3.event.scale + ")"); // not in d3 v4
      g.attr("transform", d3.event.transform); // updated for d3 v4
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
  :root {
    font-size: 62.5%;
  }

  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    padding-top: 30px;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

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

  .us-border {
    fill: none;
    stroke:black;
  }

  text {
    font-weight: 500;
    font-size: 18px;
    color: #081f2c;
  }

  div.tooltip { 
    position: absolute;     
    text-align: left;     
    padding: 12px;       
    font: 12px sans-serif;
    color: rgb(247, 247, 247);  
    background: rgb(0, 178, 160);; 
    border: 0px;        
    pointer-events: none;
    border-radius: 10%;     
  }

  div#banner.row {
    margin-right: 0px;
    margin-left: 0px;
  }

  #banner {
    display: flex;
    justify-content: center;
    padding-top: 1.5rem;
    background-color:rgb(0, 178, 160); 
    width: 100%;
    height: 6rem;
    color: rgb(247, 247, 247);
  }

  #mapwrapper {
    height: 100%;
    width: 100%
  }

  #map {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }

  #maprow {
    width: 100%;
    height: 90%;
  }

  #sidebar {
    position: relative;
    display: flex;
    flex-direction: column;
    background-color: rgb(0, 178, 160);
    margin-right: 100px;
    color: rgb(247, 247, 247);
    padding-left: 0px;
    padding-right: 0px;
    #info {
        position: relative;
        margin-top: 20px;
        width: 100%;
    }
  }

  #scrollable {
    overflow-y: auto;
    height: 100%;
  }

  #countyList {
    margin-top: 20px;
    text-align: left;
    width: 90%;
    height: 90px;
    line-height: 90px;
    padding-top: 1rem;
    #countyListItem {
        border-bottom: 1px solid #FFFFFF;
        width: 100%;
        height: 80px;
        line-height: 80px;
    }
  }

  #townshipDetail {
    font-size: 1.5rem;
    line-height: 1.5rem;
  }

  .shadow {
    -moz-box-shadow:    3px 3px 5px 6px #ccc;
    -webkit-box-shadow: 3px 3px 5px 6px #ccc;
    box-shadow:         3px 3px 5px 6px #ccc;
  }

  .loader {
    border: 16px solid #f3f3f3; /* Light grey */
    border-top: 16px solid rgb(0, 178, 160);
    border-radius: 50%;
    width: 120px;
    height: 120px;
    animation: spin 2s linear infinite;
    display: inline-block;

  }

  #selection {
    padding-top: 1.5rem;
    background-color: rgb(1,80,90);
    width: 100%;
    height: 6rem;
    font-size: 1.6rem;
    
  }


  .centered {
      position: fixed;
      top: 45%;
      left: 50%;
      transform: translate(-50%, -50%);
  }

  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  @media only screen and (min-width: 768px) {
      #sidebar {
          height: 80vh;
      }
      .wrapper-right {
          margin-top: 80px;
      }
  }
  @media only screen and (min-width:1440px) {
      #sidebar {
          width: 350px;
          max-width: 350px;
          flex: auto;
      }
      #dashboard-content {
          width: calc(100% — 350px);
          max-width: calc(100% — 350px);
          flex: auto;
      }
  }
</style>