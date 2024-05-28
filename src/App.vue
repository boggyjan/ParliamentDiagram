<template>
  <div class="wrapper">
    <header class="title">
      <h1>
        <span class="material-symbols-rounded">
          donut_large
        </span>
        國會席次分布圖產生器
      </h1>
    </header>
    <main>
      <div class="parties">
        <h2>
          <span class="material-symbols-rounded">
            groups
          </span>
          政黨
        </h2>
        <div
          v-for="(party, idx) in parties"
          :key="`party_${idx}`"
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
              <div>線條粗度</div>
              <input
                v-model.number="party.border_size"
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
              <div>線條顏色</div>
              <input
                v-model="party.border_color"
                type="color"
              >
            </div>
          </div>
          <div
            class="del-btn"
            @click="delParty(idx)"
          >
            <span class="material-symbols-rounded">
              delete_forever
            </span>
            移除
          </div>
        </div>

        <button
          type="button"
          @click="addParty()"
        >
          <span class="material-symbols-rounded">
            group_add
          </span>
          新增政黨
        </button>
      </div>

      <div class="result">
        <h2>
          <span class="material-symbols-rounded">
            image
          </span>
          產生結果
        </h2>

        <div class="result-svg">
          <ParliamentDiagram
            :denser-rows="false"
            :parties="parties"
          />
          <div class="rewritten">
            國會席次分布圖由此
            <a href="https://github.com/slashme/parliamentdiagram/blob/master/newarch.py" target="_blank">API</a>
            改寫
          </div>
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
    name: '政黨名稱',
    nb_seats: 1,
    color: '#F44ED5',
    border_size: 0,
    border_color: '#000000'
  })
}

function delParty (idx: number) {
  parties.value.splice(idx, 1)
}
</script>

<style lang="scss">
body {
  margin: 0;
  background: #111;
  background-image: linear-gradient(0, #111f, #1117), url(https://picsum.photos/id/723/800/600);
  background-size: 100vw 75vw;
  background-position: top center;
  background-repeat: no-repeat;
  color: #fafafa;
  font-size: 16px;
  font-family: sans-serif;
}

a {
  color: currentColor;
}

h1 {
  text-align: center;
}

h1, h2 {
  font-weight: 600;
}

button,
input {
  vertical-align: middle;
}

button {
  display: block;
  box-sizing: border-box;
  appearance: none;
  width: 100%;
  margin: 1rem 0;
  padding: 1rem;
  background: #fff2;
  border: 0;
  border-radius: 100px;
  color: #fff;
  cursor: pointer;
  font-size: 16px;
  outline: 0;
}

input[type=text],
input[type=number] {
  display: block;
  box-sizing: border-box;
  appearance: none;
  width: 100%;
  padding: 0.5rem 1rem;
  background: #fff2;
  border: 0;
  border-radius: 100px;
  color: #fff;
  cursor: pointer;
  font-size: 16px;
  outline: 0;
}

input[type=color] {
  display: inline-block;
  box-sizing: border-box;
  appearance: none;
  width: 4rem;
  padding: 0 0.1rem;
  background: #fff2;
  border: 0;
  border-radius: 0.4rem;
  cursor: pointer;
  outline: 0;
}

.material-symbols-rounded {
  font-variation-settings:
  'FILL' 0,
  'wght' 400,
  'GRAD' 0,
  'opsz' 24;

  vertical-align: middle;
}

.wrapper {
  padding: 20px;

  main {
    display: grid;
    grid-gap: 20px;
    grid-template-columns: 1fr 500px;

    @media (max-width: 1199px) {
      grid-template-columns: 1fr;
    }
  }

  .party-box {
    position: relative;
    margin: 20px 0;
    padding: 20px;
    background: #fff1;
    border-radius: 6px;

    .name {
      margin-bottom: 16px;
      font-weight: bold;
    }

    .props {
      display: grid;
      grid-gap: 8px 20px;
      grid-template-columns: auto auto;

      .prop-item {
        display: grid;
        grid-gap: 8px;
        grid-template-columns: auto 1fr;
        align-items: center;

        @media (max-width: 767px) {
          grid-template-columns: 1fr;
        }
      }
    }

    .del-btn {
      position: absolute;
      top: 20px;
      right: 20px;
      cursor: pointer;
    }
  }

  .result-svg {
    margin: 20px 0;

    svg {
      display: block;
      box-sizing: border-box;
      padding: 20px;
      width: 100%;
      height: auto;
      background: #fff;
      border-radius: 6px;
    }
  }

  .rewritten {
    margin-top: 2rem;
    font-size: 0.9rem;
    text-align: center;
  }
}
</style>
