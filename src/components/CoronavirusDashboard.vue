<template>
  <div class="card-body root-container">
    <div id="myModal" class="modal">
      <!-- Modal content -->
      <div class="modal-content">
        <span class="close">&times;</span>
        <h1 id="linecharthead"></h1>
        <svg id="linechartsvg"></svg>
        <div id="linetooltip"></div>
      </div>
    </div>
    <div class="sideinfo-container">
      <div id="sidebar" class="sidebar">
        <div id="countySummary" class="shadow">
          <span class="county-summary-title">County Summary</span>
          <hr/>
          <p v-for="summary in countySummary" :key="summary.fips"><strong>{{summary.countyName}}:</strong> Cases {{summary.cases}}, Deaths {{summary.deaths}}</p>
        </div>
        <div id="selection">
          <select v-model="selected" @change="selectCounty($event)">
            <option disabled value="">Choose a county</option>
            <option>Chester County</option>
            <option>Delaware County</option>
            <option>Montgomery County</option>
          </select>
        </div>
        <div id="scrollable" class="shadow">
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
    </div>
    <!-- <div class="col-md-12 col-sm-12 col-xs-12" id="banner">Vitable Coronavirus Heatmap</div> -->
    <div class="map-container">
      <div id="map-banner">
        <h1>Vitable's COVID-19 Data Tracking Tool</h1>
      </div>
      <div id="us-summary" class="one-edge-shadow">
        <div id="us-summary-detail">
          <p id="summary-label">PA. Cases</p>
          <p id="pa-summary-data">{{paTotalCases}} <span id="new-data">(+{{paNewCases}})</span></p>
        </div>
        <div id = "us-summary-detail">
          <p id="summary-label">PA. Deaths</p>
          <p id="pa-summary-data">{{paTotalDeaths}} <span id="new-data">(+{{paNewDeaths}})</span></p>
        </div>
        <div id="us-summary-detail">
          <p id="summary-label">U.S. Cases</p>
          <p id="us-summary-data">{{usTotalCases}} <span id="new-data">(+{{usNewCases}})</span></p>
        </div>
        <div id = "us-summary-detail">
          <p id="summary-label">U.S. Deaths</p>
          <p id="us-summary-data">{{usTotalDeaths}} <span id="new-data">(+{{usNewDeaths}})</span></p>
        </div>
      </div>
      <div id="mapwrapper" class="shadow">
        <div id="maprow">
          <div id="map">
            <div class="loader"></div>
            <svg id="mapsvg"/>
          </div>
        </div>
        <div id="disclaimer"><p>Data provided by <a href="https://www.nytimes.com/interactive/2020/us/coronavirus-us-cases.html">The New York Times</a></p></div>
      </div>
      <div id="map-banner">
        <p>Vitable offers in-home concierge urgent care. We'll send our providers to your home to collect your specimen and send it to our lab partner for COVID-19 testing.</p>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  /* eslint-disable */
  props: [],
  components: {},
  data() {
    return {
      selected: 'Delaware County',
      countyData: {},
      data: [],
      countySummary: [
        {
          "countyName":"Philadelphia County",
          "fips":"42101"
        },
        {
          "countyName":"Montgomery County",
          "fips": "42091"
        },
        {
          "countyName":"Delaware County",
          "fips": "42045"
        },
        {
          "countyName":"Bucks County",
          "fips": "42017"
        },
        {
          "countyName":"Chester County",
          "fips": "42029"
        },
      ],
      usTotalCases: "",
      usTotalDeaths: "",
      usNewCases: "",
      usNewDeaths: "",
      paTotalCases: "",
      paTotalDeaths: "",
      paNewCases: "",
      paNewDeaths: "",
    }
  },
  methods: {
    selectCounty(event) {
      let countyName = event.target.value;
      this.data = this.countyData[countyName];
    },
    selectCountyOnMount(countyName) {
      this.data = this.countyData[countyName]
    },
    setCountyData(countyName, data) {
      this.countyData[countyName] = data
    },
    setCountySummaries(countySummaries) {
      this.countySummary = countySummaries
    },
    setUsTotalCases(numCases) {
      this.usTotalCases = numCases;
    },
    setUsTotalDeaths(numDeaths) {
      this.usTotalDeaths = numDeaths;
    },
    setUsNewCases(numNewCases) {
      this.usNewCases = numNewCases;
    },
    setUsNewDeaths(numNewDeaths) {
      this.usNewDeaths = numNewDeaths;
    },
    setPaTotalCases(numCases) {
      this.paTotalCases = numCases;
    },
    setPaTotalDeaths(numDeaths) {
      this.paTotalDeaths = numDeaths;
    },
    setPaNewCases(numNewCases) {
      this.paNewCases = numNewCases;
    },
    setPaNewDeaths(numNewDeaths) {
      this.paNewDeaths = numNewDeaths;
    }
  },
  mounted: async function() {
    var chesterCountyCsv = "data:application/octet-stream;charset=utf-8,ID%2CValue%2CC%0AWest%20Grove%20Borough%2C0%2C0%0ANew%20London%20Township%2C0%2C0%0AHoney%20Brook%20Borough%2C0%2C0%0AEast%20Fallowfield%20Township%2C1%2C0%0ACaln%20Township%2C2%2C0%0AValley%20Township%2C0%2C0%0AElk%20Township%2C0%2C0%0AOxford%20Borough%2C1%2C0%0AWillistown%20Township%2C3%2C0%0AAvondale%20Borough%2C0%2C0%0AWest%20Fallowfield%20Township%2C0%2C0%0AAtglen%20Borough%2C0%2C0%0AFranklin%20Township%2C0%2C0%0APhoenixville%20Borough%2C0%2C0%0AParkesburg%20Borough%2C0%2C0%0ASouth%20Coventry%20Township%2C0%2C0%0AWest%20Nottingham%20Township%2C0%2C0%0AWest%20Marlborough%20Township%2C0%2C0%0ASpring%20City%20Borough%2C1%2C0%0ALondon%20Britain%20Township%2C1%2C0%0AEast%20Nantmeal%20Township%2C0%2C0%0AWest%20Caln%20Township%2C1%2C0%0AEast%20Caln%20Township%2C0%2C0%0ASchuylkill%20Township%2C0%2C0%0ACity%20of%20Coatesville%2C0%2C0%0AThornbury%20Township%2C0%2C0%0AEast%20Goshen%20Township%2C5%2C0%0AWest%20Pikeland%20Township%2C3%2C0%0AKennett%20Township%2C0%2C0%0AEast%20Marlborough%20Township%2C2%2C0%0AWest%20Chester%20Borough%2C7%2C0%0AEast%20Brandywine%20Township%2C1%2C0%0ADowningtown%20Borough%2C1%2C0%0AWest%20Brandywine%20Township%2C0%2C0%0AEast%20Bradford%20Township%2C2%2C0%0AWallace%20Township%2C0%2C0%0AEast%20Whiteland%20Township%2C8%2C0%0AEast%20Coventry%20Township%2C1%2C0%0ABirmingham%20Township%2C2%2C0%0ANew%20Garden%20Township%2C1%2C0%0ANorth%20Coventry%20Township%2C1%2C0%0ANewlin%20Township%2C0%2C0%0AWest%20Goshen%20Township%2C3%2C0%0APenn%20Township%2C0%2C0%0AWest%20Whiteland%20Township%2C4%2C0%0APennsbury%20Township%2C2%2C0%0AEast%20Pikeland%20Township%2C3%2C0%0AEast%20Vincent%20Township%2C3%2C0%0AUpper%20Oxford%20Township%2C1%2C0%0AWest%20Vincent%20Township%2C1%2C0%0AHoney%20Brook%20Township%2C2%2C0%0AUpper%20Uwchlan%20Township%2C3%2C0%0AWest%20Bradford%20Township%2C1%2C0%0AUwchlan%20Township%2C8%2C0%0ALondonderry%20Township%2C0%2C0%0ASouth%20Coatesville%20Borough%2C0%2C0%0ALower%20Oxford%20Township%2C0%2C0%0AEasttown%20Township%2C9%2C0%0AWesttown%20Township%2C0%2C0%0ALondon%20Grove%20Township%2C0%2C0%0AKennett%20Square%20Borough%2C1%2C0%0AEast%20Nottingham%20Township%2C1%2C0%0AMalvern%20Borough%2C1%2C0%0ACharlestown%20Township%2C2%2C0%0ATredyffrin%20Township%2C1%2C0%0AModena%20Borough%2C0%2C0%0AWest%20Nantmeal%20Township%2C0%2C0%0AWarwick%20Township%2C0%2C0%0AElverson%20Borough%2C0%2C0%0APocopson%20Township%2C0%2C0%0AHighland%20Township%2C0%2C0%0ASadsbury%20Township%2C0%2C0%0AWest%20Sadsbury%20Township%2C0%2C0";
    var delawareCountyCsv = "data:application/octet-stream;charset=utf-8,ID%2CValue%2CC%0AYeadon%20Borough%2C2%2C0%0AAldan%20Borough%2C1%2C0%0ATinicum%20Township%2C0%2C0%0ARose%20Valley%20Borough%2C2%2C0%0ALower%20Chichester%20Township%2C0%2C0%0ALansdowne%20Borough%2C1%2C0%0ARidley%20Township%2C6%2C0%0AProspect%20Park%20Borough%2C1%2C0%0AHaverford%20Township%2C21%2C0%0AChadds%20Ford%20Township%2C1%2C0%0ARidley%20Park%20Borough%2C4%2C1%0ASwarthmore%20Borough%2C4%2C0%0AUpland%20Borough%2C0%2C0%0AEdgmont%20Township%2C0%2C0%0AMarcus%20Hook%20Borough%2C0%2C0%0AParkside%20Borough%2C0%2C0%0AMillbourne%20Borough%2C1%2C0%0AUpper%20Chichester%20Township%2C5%2C0%0ACollingdale%20Borough%2C1%2C0%0AColwyn%20Borough%2C0%2C0%0AGlenolden%20Borough%2C0%2C0%0AChester%20Heights%20Borough%2C0%2C0%0AUpper%20Providence%20Township%2C4%2C0%0AMarple%20Township%2C11%2C1%0ANorwood%20Borough%2C1%2C0%0AClifton%20Heights%20Borough%2C3%2C0%0AAston%20Township%2C5%2C0%0AChester%20Township%2C1%2C0%0ASpringfield%20Township%2C7%2C0%0ARutledge%20Borough%2C0%2C0%0ADarby%20Township%2C0%2C0%0AConcord%20Township%2C4%2C0%0AThornbury%20Township%2C5%2C0%0AMorton%20Borough%2C3%2C0%0AMiddletown%20Township%2C10%2C1%0ADarby%20Borough%2C1%2C0%0AEddystone%20Borough%2C1%2C0%0ABrookhaven%20Borough%2C1%2C0%0ARadnor%20Township%2C21%2C0%0ATrainer%20Borough%2C1%2C0%0ANewtown%20Township%2C11%2C0%0AEast%20Lansdowne%20Borough%2C2%2C0%0AFolcroft%20Borough%2C0%2C0%0AUpper%20Darby%20Township%2C27%2C0%0ASharon%20Hill%20Borough%2C12%2C0%0ABethel%20Township%2C2%2C0%0AMedia%20Borough%2C2%2C0%0AChester%20City%2C3%2C0%0ANether%20Providence%20Township%2C9%2C0";
    var montgomeryCountyCsv = "data:application/octet-stream;charset=utf-8,ID%2CValue%2CC%0AUpper%20Merion%20Township%2C13%2C0%0ACheltenham%20Township%2C28%2C1%0ABridgeport%20Borough%2C0%2C0%0AJenkintown%20Borough%2C2%2C0%0AConshohocken%20Borough%2C5%2C0%0ARockledge%20Borough%2C0%2C0%0AWest%20Conshohocken%20Borough%2C0%2C0%0ALower%20Merion%20Township%2C68%2C0%0AUpper%20Hanover%20Township%2C1%2C0%0AEast%20Greenville%20Borough%2C0%2C0%0APennsburg%20Borough%2C0%2C0%0AWest%20Pottsgrove%20Township%2C1%2C0%0AMontgomery%20Township%2C6%2C0%0ATowamencin%20Township%2C4%2C0%0APottstown%20Borough%2C5%2C0%0ASchwenksville%20Borough%2C2%2C0%0APerkiomen%20Township%2C4%2C0%0ALansdale%20Borough%2C4%2C0%0ASkippack%20Township%2C6%2C0%0AUpper%20Gwynedd%20Township%2C4%2C0%0AHorsham%20Township%2C7%2C0%0AWorcester%20Township%2C10%2C0%0ANarberth%20Borough%2C2%2C0%0ALimerick%20Township%2C8%2C0%0ARoyersford%20Borough%2C1%2C0%0AWest%20Norriton%20Township%2C5%2C0%0AAbington%20Township%2C31%2C2%0AUpper%20Providence%20Township%2C11%2C0%0ANorth%20Wales%20Borough%2C0%2C0%0ALower%20Gwynedd%20Township%2C9%2C0%0ATrappe%20Borough%2C1%2C0%0ACollegeville%20Borough%2C3%2C0%0ALower%20Providence%20Township%2C27%2C0%0AWhitpain%20Township%2C10%2C0%0AUpper%20Moreland%20Township%2C4%2C0%0AHatboro%20Borough%2C1%2C0%0AUpper%20Dublin%20Township%2C14%2C0%0AEast%20Norriton%20Township%2C6%2C0%0AAmbler%20Borough%2C5%2C0%0ALower%20Moreland%20Township%2C10%2C0%0ABryn%20Athyn%20Borough%2C0%2C0%0ADouglass%20Township%2C1%2C0%0AMarlborough%20Township%2C0%2C0%0ARed%20Hill%20Borough%2C0%2C0%0ANew%20Hanover%20Township%2C2%2C0%0ASalford%20Township%2C1%2C0%0AUpper%20Frederick%20Township%2C3%2C0%0AGreen%20Lane%20Borough%2C0%2C0%0AFranconia%20Township%2C0%2C0%0AUpper%20Salford%20Township%2C0%2C0%0ATelford%20Borough%2C0%2C0%0ASouderton%20Borough%2C3%2C0%0ALower%20Frederick%20Township%2C0%2C0%0AHatfield%20Township%2C2%2C0%0AUpper%20Pottsgrove%20Township%2C1%2C0%0ALower%20Salford%20Township%2C4%2C0%0AHatfield%20Borough%2C0%2C0%0ALower%20Pottsgrove%20Township%2C2%2C0%0AWhitemarsh%20Township%2C9%2C1%0APlymouth%20Township%2C9%2C0%0ANorristown%20Borough%2C4%2C0%0ASpringfield%20Township%2C12%2C0";
    var svg = d3.select("#mapsvg").style("margin-top","0px").style("display","none").style("background-color","#D2D2D2");
    var path = d3.geoPath();
    var tooltipHeight = 60;
    var tooltipWidth = 200;

    var zoom = d3.zoom().on("zoom", zoomed);

    var g = svg.append("g");

    var component = this;

    d3.csv(chesterCountyCsv).then(function(d) {
      component.setCountyData("Chester County", d);
    })
    d3.csv(delawareCountyCsv).then(function(d) {
      component.setCountyData("Delaware County", d);
      component.selectCountyOnMount("Delaware County");
    })
    d3.csv(montgomeryCountyCsv).then(function(d) {
      component.setCountyData("Montgomery County", d);
    });

    var countySummaries = [
      {
        "countyName":"Philadelphia County",
        "fips":"42101"
      },
      {
        "countyName":"Montgomery County",
        "fips": "42091"
      },
      {
        "countyName":"Delaware County",
        "fips": "42045"
      },
      {
        "countyName":"Bucks County",
        "fips": "42017"
      },
      {
        "countyName":"Chester County",
        "fips": "42029"
      },
    ];


    svg.call(zoom).call(zoom.transform, d3.zoomIdentity.translate(-3600,-800).scale(5)); // delete this line to disable free zooming
    const stateDataByFips = {};
    d3.csv("https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv").then(function(stateData) {
      stateData = stateData.sort(function(a,b) {
        a = new Date(a.date);
        b = new Date(b.date);
        return a < b ? 1 : -1;
      });

      let usTotalCases = 0;
      let usTotalDeaths = 0;
      let usCasesYesterday = 0;
      let usDeathsYesterday = 0;
      let newCases = 0;
      let newDeaths = 0;
      stateData.forEach((el) => {
        if(stateDataByFips[el.fips] == null) {
            stateDataByFips[el.fips]=[el];
          } else {
            stateDataByFips[el.fips].push(el);
          }
      })
      Object.values(stateDataByFips).forEach((el)=> {
        usTotalCases = usTotalCases + parseInt(el[0].cases);
        usTotalDeaths = usTotalDeaths + parseInt(el[0].deaths);
        usCasesYesterday = usCasesYesterday + parseInt(el[1].cases);
        usDeathsYesterday = usDeathsYesterday + parseInt(el[1].deaths);
      })
      newCases = usTotalCases - usCasesYesterday;
      newDeaths = usTotalDeaths - usDeathsYesterday;
      component.setUsTotalCases(usTotalCases);
      component.setUsTotalDeaths(usTotalDeaths);
      component.setUsNewCases(newCases);
      component.setUsNewDeaths(newDeaths);

      let paTotalCases = stateDataByFips["01"][0].cases;
      let paCasesYesterday = stateDataByFips["01"][1].cases;
      let paTotalDeaths = stateDataByFips["01"][0].deaths;
      let paDeathsYesterday = stateDataByFips["01"][1].deaths;
      let paNewCases = parseInt(paTotalCases) - parseInt(paCasesYesterday);
      let paNewDeaths = parseInt(paTotalDeaths) - parseInt(paDeathsYesterday);
      component.setPaTotalCases(paTotalCases);
      component.setPaTotalDeaths(paTotalDeaths);
      component.setPaNewCases(paNewCases);
      component.setPaNewDeaths(paNewDeaths);

    }); 
    const countyDataByFips = {}
    const stateIds = []
    var tooltip = d3.select(".card-body").append("div")
      .attr("class", "tooltip")
      .style("opacity", 0)
      .style("height", tooltipHeight)
      .style("width", tooltipWidth);
    var usStates =d3.json("https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json").then(function(us) {
      us.objects.states.geometries.forEach(geometry => {
        stateIds[geometry.id] = geometry.properties.name;
      })

      d3.csv("https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties.csv").then(function(covidData) {
        covidData = covidData.sort(function(a,b) {
          a = new Date(a.date);
          b = new Date(b.date);
          return a < b ? 1 : -1;
        });
        covidData.forEach((el) => {
          if(countyDataByFips[el.fips] == null) {
            countyDataByFips[el.fips]=[el];
          } else {
            countyDataByFips[el.fips].push(el);
          }
          if(el.county == "New York City" || el.county == "Kansas City") {
            if (countyDataByFips[el.county] == null) {
              countyDataByFips[el.county] = [el];
            } else {
              countyDataByFips[el.county].push(el);
            }
          }
        });
        countySummaries.forEach(countySum => {
          countySum["cases"] = countyDataByFips[countySum.fips][0].cases;
          countySum["deaths"] = countyDataByFips[countySum.fips][0].deaths;
        })
        component.setCountySummaries(countySummaries);
        
        var extent = d3.extent(covidData, function(d) {return parseInt(d.cases);})
        // extent[1] = extent[1] * .15
        extent[0] = 1;
        var colorScale = d3.scaleLog()
          .domain(extent)
          .range(["#FFFFFF", '#710019'])
        console.log("extent " + extent);
        us.objects.counties.geometries.forEach(function(d) {
          
          d.properties.id = d.id;
          d.properties.stateId = d.id.toString().slice(0,2);
          d.properties.stateName = stateIds[d.properties.stateId];
          let exceptional = getCaseDataForExceptions(d.properties.id);
          let caseData;
          let newCases;
          let newDeaths;
          let color;
          if (exceptional != null) {
            caseData = countyDataByFips[exceptional[0]] != null ? countyDataByFips[exceptional[0]] : null;
          } else {
            if (d.properties.id == "36029") {
              console.log("Erie data" + console.log(countyDataByFips[d.properties.id]))
            }
            caseData = countyDataByFips[d.properties.id] != null ? countyDataByFips[d.properties.id] : null;
          }
          if (caseData != null) {
            let casesToday = caseData[0].cases;
            let deathsToday = caseData[0].deaths;
            let casesYesterday = caseData[1] != null ? caseData[1].cases : 0;
            let deathsYesterday = caseData[1] != null ? caseData[1].deaths : 0;

            newCases = parseInt(casesToday) - parseInt(casesYesterday)
            newDeaths = parseInt(deathsToday) - parseInt(deathsYesterday)
          } else {
            newCases = 0;
            newDeaths = 0;
          }
          color = colorScale(caseData != null ? parseInt(caseData[0].cases) : 0);
          d.properties.caseData = caseData;
          d.properties.color = color;
          d.properties.newCases = newCases;
          d.properties.newDeaths = newDeaths;
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
        let numCases = dataProperties.caseData !== null ? dataProperties.caseData[0].cases : "no cases reported";
        let numDeaths = dataProperties.caseData !== null ? dataProperties.caseData[0].deaths : "no deaths reported";
        let newCases = dataProperties.newCases;
        let newDeaths = dataProperties.newDeaths;
        let exceptional = getCaseDataForExceptions(countyId);
        if (exceptional == null) {
          tooltip
              .style("opacity", 0.9);
              tooltip.html("<p><strong>" + dataProperties.name + ", " + dataProperties.stateName + "</strong><p>" +
              "<p>Cases: " + numCases + "</p>" +
              "<p>Deaths: " + numDeaths + "</p>" +
              "<p>New cases: " + newCases + "</p>" +
              "<p>New deaths: " + newDeaths + "</p>")
              // .style("left", (d3.event.pageX - offsetLeft - tooltipWidth*.8) + "px")
              // .style("top", (d3.event.pageY - offsetTop - tooltipHeight*1.5) + "px");
              .style("left",d3.event.pageX + "px")
              .style("top", d3.event.pageY + "px");
        } else {
          tooltip
              .style("opacity", 0.9);
              tooltip.html("<p><strong>" + dataProperties.name + ", " + dataProperties.stateName + "</strong><p>" +
              "<p>Cases for " + exceptional[0] + ": " + numCases + "</p>" +
              "<p>Deaths for " + exceptional[0] + ": " + numDeaths + "</p>" +
              "<p>New cases: " + newCases + "</p>" +
              "<p>New deaths: " + newDeaths + "</p>")
              // .style("left", (d3.event.pageX - offsetLeft - tooltipWidth*0.8) + "px")
              // .style("top", (d3.event.pageY - offsetTop - tooltipHeight*1.5) + "px");
              .style("left",d3.event.pageX + "px")
              .style("top", d3.event.pageY + "px");
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
      d3.selectAll(".counties")
        .style("fill", function(d) {
          return d.properties.color;
        })
      svg.style("display", "block");
      d3.select(".loader").style("display","none");
    }

    /* Line chart business */
    var modal = d3.select("#myModal");
    var close = d3.select(".close").on("click", ()=>{
      modal.style("display","none");
      modal.selectAll("rect").remove();
      modal.selectAll("#axis").remove();
      modal.selectAll("circle").remove();
      modal.selectAll("path").remove();
    });
    const lineChartSvg = d3.select("#linechartsvg");
    var svgWidth;
    var svgHeight;
    var lcMargin = {top: 0, right: 20, bottom: 80, left: 20}
    var lineChartWidth;
    var lineChartHeight;
    var x;
    var y;
    var line;
    var deathLine;
    var lineChart = lineChartSvg.append('g').attr("id","lineChart");
    const parseTime = d3.timeParse("%Y-%m-%d");
    const bisectDate = d3.bisector(d3.descending).left;

    function clicked() {
      console.log("clicked");
      let dataProperties = d3.select(this).data()[0].properties;
      console.log("data properties " + JSON.stringify(dataProperties));
      let data = []
      console.log("data" + JSON.stringify(data));
      dataProperties.caseData.forEach((d) => {
        data.push({
          "date": parseTime(d.date),
          "cases": +d.cases,
          "deaths": + d.deaths
        })
      });
      d3.select("#linecharthead").html(dataProperties.name + ", " + dataProperties.stateName);
      console.log("data update " + JSON.stringify(data));
      modal.style("display",null);
      svgWidth = +lineChartSvg.style("width").replace("px","");
      console.log("svg width " + svgWidth)
      svgHeight = +lineChartSvg.style("height").replace("px","");
      console.log("svg height " +svgHeight);
      lineChartWidth = svgWidth - lcMargin.left - lcMargin.right;
      console.log("line chart width " + lineChartWidth);
      lineChartHeight = svgHeight - lcMargin.top - lcMargin.bottom;
      console.log("lineChartHeight " + lineChartHeight);
      lineChart
        .attr('transform', 'translate(' + lcMargin.left + ',' + lcMargin.top + ')');
      x = d3.scaleTime().domain(d3.extent(data.map(d => d.date))).range([0, lineChartWidth]);
      let ex = d3.extent(data.map(d => d.cases))
      y = d3.scaleLinear().domain([0,ex[1]]).range([lineChartHeight, 0]);
      line = d3.line().x(d => x(d.date)).y(d => y(d.cases));
      deathLine = d3.line().x(d => x(d.date)).y(d => y(d.deaths));
      updateLineChart(data);
      
    }

    function updateLineChart(data) {
      lineChart
        .append("path")
        .data([data])
        .attr("class", "line")
        .attr("d",line)
      lineChart
        .append("path")
        .data([data])
        .attr("class", "deathline")
        .attr("d",deathLine)

      lineChartSvg
        .append("g")
        .attr("id","axis")
        .attr("transform", "translate(" + lcMargin.left + "," + (svgHeight - lcMargin.bottom) + ")")
        .call(d3.axisBottom(x).ticks(5));

      // add y axis
      lineChartSvg.append("g").call(d3.axisLeft(y).ticks(5)).attr("id","axis").attr("transform", "translate(" + lcMargin.left + ","+ lcMargin.top + ")");

        // create focus area
      const focus = lineChart
        .append("g")
        .attr("class", "focus")
        .style("display", "none");

      focus
        .append("line")
        .attr("class", "x-hover-line hover-line")
        .attr("y1", 0)
        .attr("y2", lineChartHeight);

      focus
        .append("line")
        .attr("class", "y-hover-line hover-line")
        .attr("x1", 0)
        .attr("x2", lineChartSvg);

      focus.append("circle").attr("r", 6);

      // backdrop for tooltip
      focus
        .append("rect")
        .attr("class", "tooltip")
        .attr("width", "160")
        .attr("height", "60")
        .attr("y", "-4em")
        .attr("x", "-80")
        .attr("rx", "5")
        .attr("ry", "5");

      const focusText = focus.append("text").attr("class", "textBox");

      focusText
        .append("tspan")
        .attr("class", "tooltip-text")
        .attr("x", "-75")
        .attr("dy", "-1em");

      focusText
        .append("tspan")
        .attr("class", "tooltip-text")
        .attr("x", "-75")
        .attr("dy", "-1em");
      
      focusText
        .append("tspan")
        .attr("class", "tooltip-text")
        .attr("x", "-75")
        .attr("dy", "-1em");

      // append area for pulling mouseovers and calling function to display lines
      // and text boxes
      lineChart
        .append("rect")
        .attr("transform", "translate(" + 0 + "," + 0 + ")")
        .attr("class", "overlay")
        .attr("fill", "transparent")
        .attr("width", lineChartWidth)
        .attr("height", lineChartHeight)
        .on("mouseover", function() {
          // focus.style("display", null);
        })
        .on("mouseout", function() {
          focus.style("display", "none");
        })
        .on("mousemove", mousemove);

      function mousemove() {
        const x0 = x.invert(d3.mouse(this)[0]);
        const i = bisectDate(data.map(d => d.date), x0);
        const d0 = data[i - 1];
        const d1 = data[i];
        const d = x0 - d0.date < d1.date - x0 ? d1 : d0;
        focus.style("display",null);
        focus.attr(
          "transform",
          "translate(" + x(d.date) + "," + y(d.cases) + ")"
        );
        focus
          .selectAll("tspan")
          .nodes()[2].textContent = `Date: ${d.date.toDateString()}`;
        focus.selectAll("tspan").nodes()[1].textContent = `Cases: ${d.cases}`;
        focus.selectAll("tspan").nodes()[0].textContent = `Deaths: ${d.deaths}`;
        
        focus.select(".x-hover-line").attr("y2", lineChartHeight - y(d.cases));
        focus.select(".y-hover-line").attr("x2", -x(d.date));
      }
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

<style lang="sass">
  @import "../styles/colors"
  root
    font-size: 62.5%

  .root-container
    display: flex
    flex-direction: row
    justify-content: space-around

  .sideinfo-container
    display: flex
    flex-direction: column
    justify-content: space-between
    height: 100%
    width: 34%
    padding: 3%
    border: 0.5px solid #f6f6f6
    border-radius: 2px

  .map-container
    display: flex
    flex-direction: column
    flex: 1
    padding: 3%
    width: 66%
    height: 100%
    border: 0.5px solid #f6f6f6
    border-radius: 2px

  .county-summary-title
    color: white
    font-size: 1.5em
    font-weight: bold
    display: flex
    justify-content: center

  hr
    display: block
    height: 1px
    border: 0
    border-top: 1px solid #FFFFFF
    margin: .5em 0
    padding: 0

  #county
    fill: #D2D2D2
    &:hover
      fill: red
      opacity: 0.5

  .state-borders
    fill: none

  .county-borders
    fill: none
    stroke: #121212
    stroke-width: 0.5px
    stroke-linejoin: round
    stroke-linecap: round
    pointer-events: none

  .us-border
    fill: none
    stroke: black

  div.tooltip
    position: absolute
    text-align: left
    padding: 12px
    font: 12px sans-serif
    color: rgb(247, 247, 247)
    background: rgb(0, 178, 160)
    border: 1px
    border-style: solid
    border-color: $primary
    pointer-events: none
    border-radius: 10%

  #map-banner
    text-align: center
    background-color: $highlight
    width: 100%
    color: white
    h1
      font-size: 25px
    p
      font-size: 18px
      font-weight: bold

  #map
    display: flex
    justify-content: center
    padding: 1%
    width: 100%
    height: 100%

  #mapsvg
    width: 90%
    height: 30em

  #disclaimer
    text-align: left
    padding: 1%
    width: 100%
    font-size: 16px

  #sidebar
    display: flex
    flex-direction: column
    height: 37.5em
    background-color: white
    color: white
    #info
        text-align: center
        position: relative
        width: 100%

  #countySummary
    padding: 3%
    text-align: left
    font-size: 1em
    background-color: $primary
    margin-bottom: 5%
    border-radius: 2px

  #scrollable
    background-color: $highlight
    overflow-y: auto
    height: 100%
    border-radius: 4px

  #countyList
    text-align: center
    width: 90%
    height: 100%
    line-height: 90px
    padding-top: 1rem
    background-color: $highlight
    #countyListItem
        border-bottom: 1px solid #FFFFFF
        width: 100%
        height: 80px
        line-height: 80px

  #townshipDetail
    font-size: 1.5rem
    line-height: 1.5rem

  .shadow
    -moz-box-shadow:    3px 3px 5px 6px #ccc
    -webkit-box-shadow: 3px 3px 5px 6px #ccc
    box-shadow:         3px 3px 5px 6px #ccc

  .loader
    border: 16px solid #f3f3f3 /* Light grey */
    border-top: 16px solid rgb(0, 178, 160)
    border-radius: 50%
    width: 120px
    height: 120px
    animation: spin 2s linear infinite
    display: inline-block

  #selection
    background-color: $primary
    width: 100%
    font-size: 1em
    margin-bottom: 5%

  .centered
      position: fixed
      top: 45%
      left: 50%
      transform: translate(-50%, -50%)
  
  #us-summary
    display: flex;
    flex-direction: row

  #us-summary-detail
    flex-grow: 1;
    padding-inline-start: 1em;
    padding-inline-end: 2em;
    border-right-style: solid;
    border-left-style: solid;
    border-color: #D2D2D2;

  #new-data
    font-size: 1.25rem;
    color: #A2A2A2;

  #summary-label
    font-size: 0.9rem;
    margin-block-end: 0.1em;
    margin-block-start: 0.1em;
    color: rgb(47,47,47);
  #us-summary-data
    font-size: 3rem;
    margin-block-end: 0.1em;
    margin-block-start: 0.1em;
    font-size: 3rem;
    font-weight: 600;
    color: black;

  #pa-summary-data
    font-size: 3rem;
    margin-block-end: 0.1em;
    margin-block-start: 0.1em;
    font-size: 3rem;
    font-weight: 600;
    color: rgb(165,0,14);

  .one-edge-shadow 
    -webkit-box-shadow: 0 8px 6px -6px black;
    -moz-box-shadow: 0 8px 6px -6px black;
    box-shadow: 0 8px 6px -6px black;

  .modal
    position: fixed
    z-index: 1
    transform: translate(-50%,-50%)
    left: 50%
    top: 50%
    width: 100%
    height: 100vh
    overflow: auto
    background-color: rgb(0,0,0)
    background-color: rgba(0,0,0,0.4)

  .modal-content
    position: fixed
    left: 50%
    top: 50%
    transform: translate(-50%, -50%)
    height: 40%
    background-color: #fefefe
    padding: 20px
    border: 1px solid #888
    width: 40%
  
  #linechartsvg
    height: 100%
    width: 100%
    overflow: visible

  .line
    fill: none
    stroke: #03dac5
    stroke-width: 3px
    text-anchor: end

  .deathline
    fill: none
    stroke: rgb(165,0,14)
    stroke-width: 3px
    text-anchor: end

  .close 
    color: #aaa
    float: right
    font-size: 28px
    font-weight: bold

  .close
    &:focus
      color: black
      text-decoration: none
      cursor: pointer
    &:hover
      color: black
      text-decoration: none
      cursor: pointer

  .tooltip
    fill: $highlight
    opacity: 0.9

  .tooltip-text
    fill: #ffffff
    opacity: 0.87

  .focus circle
    fill: #f1f3f3;
    stroke: $primary
    stroke-width: 5px;

  .hover-line 
    stroke: $primary
    stroke-width: 2px;
    stroke-dasharray: 2, 4;
    z-index: 5;

  #linecharthead
    margin-block-start: 0em
    margin-block-end: 0em
    text-align: center
  
    

  @keyframes spin
    0% {transform: rotate(0deg)}
    100% {transform: rotate(360deg)}

  @media only screen and (max-width: 576px)
    overflow-x: hidden
    h1
      font-size: 2rem
    .map-container
      width: 100%
    .root-container
      display: flex
      flex-direction: column-reverse
      justify-content: space-around
    .sideinfo-container
      width: 100%
    #sidebar
      height: 30em
    #mapwrapper
      height: 30em
    #mapsvg
      width: 90%
      height: 23em
    .modal-content
      width: 80%

</style>
