<template>
  <div class="wrapper">
    <header class="title">
      <h1>席次分布圖產生器</h1>
    </header>
    <main>
      <div class="parties">
        <h2>政黨</h2>
        <div
          v-for="(party, idx) in parties"
          class="party-box"
        >
          <div class="name">
            {{ idx + 1}}. {{ party.name || '政黨名稱' }}
          </div>

          <div class="props">
            <div class="prop-item">
              <div>政黨名稱</div>
              <input
                v-model="party.name"
                type="text"
              >
            </div>
            <div class="prop-item">
              <div>席次</div>
              <input
                v-model.number="party.nb_seats"
                type="number"
                min="0"
              >
            </div>
            <div class="prop-item">
              <div>顏色</div>
              <input
                v-model="party.color"
                type="color"
              >
            </div>
            <div class="prop-item">
              <div>線條粗度</div>
              <input
                v-model.number="party.border_size"
                type="number"
                min="0"
              >
            </div>
            <div class="prop-item">
              <div>線條顏色</div>
              <input
                v-model="party.border_color"
                type="color"
              >
            </div>
          </div>
          <button
            type="button"
            @click="delParty(idx)"
          >
            移除
          </button>
        </div>

        <button
          type="button"
          @click="addParty()"
        >
          新增政黨
        </button>
      </div>

      <div class="result">
        <h2>產生結果</h2>

        <div class="result-svg">
          <ParliamentDiagram
            :denser-rows="false"
            :parties="parties"
          />
        </div>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ParliamentDiagram from './components/ParliamentDiagram.vue'

const parties = ref([
  {
    name: '民主進步黨',
    nb_seats: 51,
    color: '#1B9431',
    border_size: 0,
    border_color: '#000000'
  },
  {
    name: '台灣民眾黨',
    nb_seats: 8,
    color: '#28C8C8',
    border_size: 0,
    border_color: '#000000'
  },
  {
    name: '無黨籍',
    nb_seats: 2,
    color: '#999999',
    border_size: 0,
    border_color: '#000000'
  },
  {
    name: '中國國民黨',
    nb_seats: 52,
    color: '#000095',
    border_size: 0,
    border_color: '#000000'
  }
])

function addParty () {
  parties.value.push({
    name: 'Party Name',
    nb_seats: 2,
    color: '#E2FED9',
    border_size: 0,
    border_color: '#000000'
  })
}

function delParty (idx) {
  parties.value.splice(idx, 1)
}
</script>

<style lang="scss">
body {
  margin: 0;
  background: #222;
  color: #fafafa;
}

.wrapper {
  padding: 20px;

  main {
    display: grid;
    grid-gap: 20px;
    grid-template-columns: 1fr auto;
  }

  .party-box {
    margin: 20px 0;
    padding: 20px;
    background: #fff1;
    border-radius: 6px;

    .name {
      margin-bottom: 10px;
      font-weight: bold;
    }

    .props {
      display: grid;
      grid-gap: 8px 20px;
      grid-template-columns: auto auto auto;

      .prop-item {
        display: grid;
        grid-gap: 8px;
        grid-template-columns: auto 1fr;
        align-items: center;
      }
    }
  }

  .result-svg {
    margin: 20px 0;
    padding: 20px;
    background: #fff;
    box-shadow: 0 0 10px #0001;
    border-radius: 6px;

    svg {
      width: 400px;
      height: auto;
      aspect-ratio: 360 / 185;
    }
  }
}
</style>
