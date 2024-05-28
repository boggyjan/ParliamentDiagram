<template>
  <div>
    <div
      ref="svgContainer"
      v-html="svgContent"
    />

    <button
      type="button"
      @click="download()"
    >
      <span class="material-symbols-rounded">
        download
      </span>
      下載SVG
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

const totals = [
  1, 15, 33, 61, 95, 138, 189, 247, 313, 388, 469, 559, 657, 762, 876,  997,
  1126, 1263, 1408, 1560, 1722, 1889, 2066, 2250, 2442, 2641, 2850, 3064,
  3289, 3519, 3759, 4005, 4261, 4522, 4794, 5071, 5358, 5652, 5953, 6263,
  6581, 6906, 7239, 7581, 7929, 8287, 8650, 9024, 9404, 9793, 10187, 10594,
  11003, 11425, 11850, 12288, 12729, 13183, 13638, 14109, 14580, 15066, 15553,
  16055, 16557, 17075, 17592, 18126, 18660, 19208, 19758, 20323, 20888, 21468,
  22050, 22645, 23243, 23853, 24467, 25094, 25723, 26364, 27011, 27667, 28329,
  29001, 29679, 30367, 31061
]

function range (start, end, step = 1) {
  let output = []
  if (typeof end === 'undefined') {
    end = start
    start = 0
  }
  for (let i = start; i < end; i += step) {
    output.push(i)
  }
  return output
}

function countDelegates (parties) {
  const result = parties.filter(party => party.nb_seats).map(party => party.nb_seats).reduce((a, b) => a + b, 0)

  if (result > totals[totals.length - 1]) {
    return 0
  }
  return result
}

function getNumberOfRows (nbDelegates) {
  for (let i of range(totals.length)) {
    if (totals[i] >= nbDelegates) {
      return i + 1
    }
  }
  // return totals.findIndex(item => item > nbDelegates) + 1
}

function getSpotsCenters (nbDelegates, nbRows, spotRadius, denseRows) {
  let discardedRows, diagramFullness

  if (denseRows) {
    [discardedRows, diagramFullness] = optimizeRows(nbDelegates, nbRows)
  } else {
    discardedRows = 0
    diagramFullness = parseFloat(nbDelegates) / totals[nbRows - 1]
  }

  const positions = []

  for (let i of range(1 + discardedRows, nbRows)) {
    // Fill the n-1 firsts rows
    addIthRowSpots(positions, nbRows, i, spotRadius, diagramFullness)
  }

  addLastRowSpots(positions, nbDelegates, nbRows, spotRadius)
  positions.sort().reverse()

  return positions
}

function optimizeRows (nbDelegates, theoriticalNbRows) {
  let handledSpots = 0
  let rowsNeeded = 0

  for (let i of range(theoriticalNbRows, 0, -1)) {
    // How many spots can we fit in each row
    // This 2 lines formula was determined by @slashme's math
    const magicNumber = 3.0 * theoriticalNbRows + 4.0 * i - 2.0
    const maxSpotInRow = Math.PI / (2 * Math.asin(2.0 / magicNumber))
    handledSpots += parseInt(maxSpotInRow)
    rowsNeeded += 1

    if (handledSpots >= nbDelegates) {
      const nbUselessRows = i - 1
      const diagramFullness = parseFloat(nbDelegates) / handledSpots
      return [nbUselessRows, diagramFullness]
    }
  }
}

function addIthRowSpots (spotsPositions, nbRows, i, spotRadius, diagramFullness) {
  // Each row can contain pi/(2asin(2/(3n+4i-2))) spots, where n is the
  // number of rows and i is the number of the current row.
  const magicNumber = 3.0 * nbRows + 4.0 * i - 2.0
  const maxSpotInRow = Math.PI / (2 * Math.asin(2.0 / magicNumber))

  // Fill the row proportionally to the "fullness" of the diagram
  const nbSpotsInIthRow = parseInt(diagramFullness * maxSpotInRow)

  // The radius of the ith row in an N-row diagram (Ri) is (3n+4*i-2)/(4n)
  const ithRowRadius = magicNumber / (4.0 * nbRows)
  appendRowSpotsPositions(spotsPositions, nbSpotsInIthRow, spotRadius, ithRowRadius)
}

function addLastRowSpots (spotsPositions, nbDelegates, nbRows, spotRadius) {
  const nbLeftoverSeats = nbDelegates - spotsPositions.length
  const lastRowRadius = (7.0 * nbRows - 2.0) / (4.0 * nbRows)
  appendRowSpotsPositions(spotsPositions, nbLeftoverSeats, spotRadius, lastRowRadius)
}

function appendRowSpotsPositions (spotsPositions, nbSeatsToPlace, spotRadius, rowRadius) {
  const sinRrr = Math.sin(spotRadius / rowRadius)

  for (let i of range(nbSeatsToPlace)) {
    let angle

    if (nbSeatsToPlace === 1) {
      angle = Math.PI / 2.0
    } else {
      angle = parseFloat(i) * (Math.PI - 2.0 * sinRrr) / (parseFloat(nbSeatsToPlace) - 1.0) + sinRrr
    }
    spotsPositions.push([
      angle,
      rowRadius * Math.cos(angle) + 1.75,
      rowRadius * Math.sin(angle)
    ])
  }
}

function writeSvgHeader (partyList) {
  return `<?xml version="1.0" encoding="UTF-8" standalone="no"?>\n
    <svg xmlns:svg="http://www.w3.org/2000/svg" xmlns="http://www.w3.org/2000/svg" version="1.1" width="360" height="${200 + partyList.length * 25}" viewBox="0 0 360 ${200 + partyList.length * 25}">\n
      <g>`
}

function writeSvgNumberOfSeats (nbSeats) {
  // Print the number of seats in the middle at the bottom.
  return `<text x="180" y="175" style="font-size:36px;font-weight:bold;text-align:center;text-anchor:middle;font-family:sans-serif">${nbSeats}</text>`
}

function writeSvgSeats (partyList, positionsList, radius) {
  let drawnSpots = 0
  let result = ''

  for (let i of range(partyList.length)) {
    // Remove illegal characters from party's name to make an svg id
    const partyName = partyList[i].name
    const sanitizedPartyName = partyName.replace(/[^a-zA-Z0-9_-]/g, '-')
    const blockId = `${i}_${sanitizedPartyName}`

    const partyNbSeats = partyList[i].nb_seats || 0
    const partyFillColor = partyList[i].color
    const partyBorderWidth = partyList[i].border_size * radius * 10
    const partyBorderColor = partyList[i].border_color

    // <g> header
    result += `<g style="fill:${partyFillColor}; stroke-width:${partyBorderWidth}; stroke:${partyBorderColor}" id="${blockId}">\n`

    // Party name in a tooltip
    result += `<title>${partyName}</title>`

    for (let j of range(drawnSpots, drawnSpots + partyNbSeats)) {
      const x = 5 + 100 * positionsList[j][1]
      const y = 5 + 100 * (1.75 - positionsList[j][2])
      const r = radius * 100 - partyBorderWidth / 2
      // <circle> element
      result += `<circle cx="${x}" cy="${y}" r="${r}"/>\n`
    }
    result += `</g>\n`
    // Close <g>
    drawnSpots += partyNbSeats
  }

  return result
}

function writePartyNames (partyList) {
  let result = ''

  for (let i = 0; i < partyList.length; i++) {
    const party = partyList[i]
    result += `<rect x="10" y="${196 + i * 25}" width="20" height="20" fill="${party.color}" stroke-width="${party.border_size}" stroke="${party.border_color}" />`
    result += `<text x="40" y="${210 + i * 25}" style="font-size:12px;font-family:sans-serif">${party.name}</text>`
  }

  return result
}

function writeSvgFooter (output) {
  return `</g>\n
    </svg>`
}

function makeResult (denserRows, parties) {
  const sumDelegates = countDelegates(parties)
  const nbRows = getNumberOfRows(sumDelegates)
  // Maximum radius of spot is 0.5/nb_rows; leave a bit of space.
  const radius = 0.4 / nbRows
  const posList = getSpotsCenters(sumDelegates, nbRows, radius, denserRows)
  return drawSvg(sumDelegates, parties, posList, radius)
}

function drawSvg (sumDelegates, parties, posList, radius) {
  let result = ''
  result += writeSvgHeader(parties)
  result += writeSvgNumberOfSeats(sumDelegates)
  result += writeSvgSeats(parties, posList, radius)
  result += writePartyNames(parties)
  result += writeSvgFooter()
  return result
}

const svgContainer = ref(null)

function download () {
  const svg = svgContainer.value.querySelector('svg')
  const data = '<?xml version="1.0" encoding="utf-8"?>' + new XMLSerializer().serializeToString(svg)
  const svgBlob = new Blob([data], { type: 'image/svg+xml;charset=utf-8' })
  const url = URL.createObjectURL(svgBlob)

  let a = document.createElement('a')
  a.setAttribute('download', 'image.svg')
  a.setAttribute('href', url)
  a.setAttribute('target', '_blank')
  a.click()
}

const props = defineProps<{
  denserRows: boolean,
  parties: array
}>()

const svgContent = computed(() => {
  return makeResult(props.denserRows, props.parties)
})
</script>
