<script lang="ts">
  import Loading from "../Loading.svelte"


  import type { CircuitType, DriverType, ResultType } from '../../utils/types'
  import fetchWikipediaImages from "../../utils/fetchWikipediaImages"
  import formatPercent from "../../utils/formatPercent"
  import getFullDriverName from "../../utils/getFullDriverName";
  import getPositionsRange from "../../utils/getPositionsRange"

  import QualToResBarChart from './QualToResBarChart.svelte'
  import QualToResMatrix from './QualToResMatrix.svelte'

  import DriverFilter from "../Filters/DriverFilter.svelte"
  import CircuitsFilter from "../Filters/CircuitsFilter.svelte"

  import getFilteredQualifyings from "./getFilteredQualifyings"

  import {
    drivers,
    latestRace,
    qualifying,
    raceIdMap,
    results,
  } from "../../stores/data"


  let circuitFilter: CircuitType | null = null
  let driverFilter: DriverType | null = null


  // $: {
  //   if(driverFilter) {
  //     const title = driverFilter.url.slice(29) //str.indexOf("http://en.wikipedia.org/wiki/") + "http://en.wikipedia.org/wiki/".length
  //     fetchWikipediaImages(title).then(console.log)
  //   }
  // }
  $: driverFullName = driverFilter ? getFullDriverName(driverFilter) : ""
  $: getHoverText = (
    qualNum: number,
    qualPosition: string,
    resultNum: number,
    resultPosition: string,
  ) => {
    const resultPositionStr = (
      resultPosition === "DNF"
      ? `did not finish (DNF) the race`
      : `finished the race in position ${resultPosition}`
    )

    return `Out of ${qualNum} qualifyings in position ${qualPosition}, ${driverFullName || "drivers"} ${resultPositionStr} a total of ${resultNum} times (${formatPercent(resultNum/qualNum)})`
  }


  //create an object that maps all the qualifying ids to the respective result
  //this is only run once when the qualifying data loads
  $: qualIdToResult = $qualifying.reduce((acc, q) => {
    acc[q.qualifyId] = $results.find( //find and set the result
      r => ( //given the result
        q.raceId === r.raceId //must have the same race id
        && q.driverId === r.driverId //must have the same driver id
      )
    )
    return acc
  }, {} as {[qualifyId: string]: ResultType})

  //filter the qualifyings based on the driver
  $: filteredQualifying = getFilteredQualifyings($qualifying, $raceIdMap, circuitFilter, driverFilter)
  $: fitleredResults = filteredQualifying.map(q => qualIdToResult[q.qualifyId]).filter(r => r)



  $: resultPositions = getPositionsRange(fitleredResults) //array of positions from results
  $: qualifyingPositions = getPositionsRange(filteredQualifying) //array of positions from qualifying

  //create a default distribution map where each position has a value of 0
  //ie { 1: 0, 2: 0, 3: 0, ... "\N": 0 }
  $: defaultDistributionMap = resultPositions.reduce((acc, p) => {
    acc[p] = 0
    return acc
  }, {})

  //2d array of qualifyings, where each row is all the qualifyings corresponding to its respective position
  //qualifyingsForPositions[qualifyingIndex] = [qualifyings for the given qualifyingIndex]
  $: qualifyingsForPositions = qualifyingPositions.map(p => filteredQualifying.filter(q => q.position === p))

  //2d array of results, where each cell is the respective result of the originating qualifying
  //resultsForPositions[qualifyingIndex] = [results for the given qualifyingIndex]
  $: resultsForPositions = qualifyingsForPositions.map(
    qRow => qRow.map(
      q => qualIdToResult[q.qualifyId]
    ).filter(r => r)
  )

  //array of distribution maps that sum the results for the given qualifying
  //distributionMaps[qualifyingIndex] = for the given qualifyingIndex, a key-value object { [position]: number of results that ended in this position  }
  $: distributionMaps = resultsForPositions.map(
    filteredResults => filteredResults.reduce((acc, r) => {
      if(acc[r.position] === undefined) { //if this is an unexpected position
        console.warn(`Encountered unexpected position ${r.position}`)
      }
      else {
        acc[r.position]++ //increment the count for this position
      }

      return acc
    }, JSON.parse(JSON.stringify(defaultDistributionMap)))
  )

</script>

<main class="qualifying-to-results">
  <header>
    <div class="img-container">
      <img src="redbull.jpg" alt="redbull"/>
      <p>Photo from <a href="https://www.formula1.com/en/latest/article.tech-tuesday-why-red-bull-could-hit-the-ground-running-in-2020.4h5ZgPepX96WB0HsianTBT.html" target="_blank" rel="noopener noreferrer">Formula 1</a></p>
    </div>
    <div class="heading-text">
      <h1>How do drivers' qualifying finishes correlate with their race results?</h1>
    </div>
  </header>

  <section>
    <div>
      <p>To determine the starting lineup for each race, F1 currently runs a <b>qualifying</b> session the day before, in which drivers compete to get the best lap times. Loosely, on the day of the race, drivers line up fastest to slowest based on their best qualifying lap times, with the fastest driver in front (aka "pole position"). During the actual race, of course, drivers constantly change positions, and the final race results are usually different from the initial lineup. Given a driver who qualified first, what are their chances of finishing the race first? Explore the data below! {#if $latestRace } (Up-to-date to the {$latestRace.date} {$latestRace.name}) {/if}</p>
    </div>

    {#if $drivers.length === 0 || $results.length === 0}
      <Loading/>
    {:else}
      <CircuitsFilter bind:circuitFilter={circuitFilter}/>
      <DriverFilter bind:driverFilter={driverFilter}/>

      <hr/>

      <QualToResBarChart
        {distributionMaps}
        {getHoverText}
        {qualifyingPositions}
        {resultPositions}
        {resultsForPositions}
      />

      <br/>
      <hr/>

      <QualToResMatrix
        {distributionMaps}
        {getHoverText}
        {qualifyingPositions}
        {resultPositions}
      />
    {/if}
  </section>
</main>

<style>
</style>
