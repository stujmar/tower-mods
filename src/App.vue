<template>
  <main>
    <h3>Tower Mods</h3>
    <div v-if="results.length > 0" class="results">
      <div v-for="(result, ndx) in results" :key="ndx" class="result-container">
        <div :class="result + ' icon'"></div>
      </div>
    </div>
    <hr>
    <div v-if="chances" style="margin-top: 16px; color: #fff; text-align: center;">
      <div>
        Result: <b>{{ chances.epicCount }} epics, {{ chances.rareCount }} rares, {{ chances.commonCount }} commons</b>
      </div>
      <div>
        Odds of this group composition: <b>{{ chances.odds }}</b>
      </div>
    </div>
    <div style="margin-top: 16px; color: #aaa; text-align: left;">
      <div>Odds of getting exactly X epics in {{ results.length }} pulls:</div>
      <div v-for="epic in epicChances" :key="epic.k">
        <span v-if="epic.k === chances.epicCount" style="font-weight:bold; color:#ff03dd;">{{ epic.k }} epics: {{ epic.odds }}</span>
        <span v-else>{{ epic.k }} epics: {{ epic.odds }}</span>
      </div>
    </div>
    <div class="buy-buttons">
      <!-- <button class="buy-button" @click="buyMods(1)">BUY 1</button> -->
      <button class="buy-button" @click="buyMods(10)">BUY 10</button>
    </div>
  </main>
</template>

<script>

export default {
  name: 'App',
  components: {},
  data() {
    return {
      currentMods: 0,
      results: [],
      showEpicOdds: false,
      chances: null,
      epicChances: [],
    }
  },
  methods: {
    buyMods(amount) {
      this.results = [];
      for (let i = 0; i < amount; i++) {
        const randomNumber = Math.random();
        if (randomNumber < 0.685) {
          this.results.push('common');
        } else if (randomNumber < 0.975) {
          this.results.push('rare');
        } else {
          this.results.push('epic');
        }
      }
      this.showEpicOdds = false;
      this.chances = this.computeChances();
      this.epicChances = this.computeEpicChances();
    },
    // Multinomial probability: P = n!/(k1!k2!k3!) * p1^k1 * p2^k2 * p3^k3
    multinomialProbability(n, counts, ps) {
      function factorial(x) {
        if (x === 0 || x === 1) return 1;
        let prod = 1;
        for (let i = 2; i <= x; i++) prod *= i;
        return prod;
      }
      let numerator = factorial(n);
      let denominator = 1;
      let prob = 1;
      for (let i = 0; i < counts.length; i++) {
        denominator *= factorial(counts[i]);
        prob *= Math.pow(ps[i], counts[i]);
      }
      return (numerator / denominator) * prob;
    },
    computeChances() {
      // Count occurrences
      const n = this.results.length;
      const epicCount = this.results.filter(r => r === 'epic').length;
      const rareCount = this.results.filter(r => r === 'rare').length;
      const commonCount = this.results.filter(r => r === 'common').length;
      // Probabilities
      const pEpic = 0.025;
      const pRare = 0.29;
      const pCommon = 0.685;
      // Multinomial probability for this group
      const prob = this.multinomialProbability(n, [commonCount, rareCount, epicCount], [pCommon, pRare, pEpic]);
      const odds = prob > 0 ? `1 in ${Math.round(1/prob).toLocaleString()}` : 'Extremely rare';
      return {
        epicCount, rareCount, commonCount,
        odds
      };
    },
    // Binomial probability for exactly k epics in n pulls
    binomialProbability(n, k, p) {
      function factorial(x) {
        if (x === 0 || x === 1) return 1;
        let prod = 1;
        for (let i = 2; i <= x; i++) prod *= i;
        return prod;
      }
      function combination(n, k) {
        return factorial(n) / (factorial(k) * factorial(n - k));
      }
      return combination(n, k) * Math.pow(p, k) * Math.pow(1 - p, n - k);
    },
    computeEpicChances() {
      const n = this.results.length;
      const pEpic = 0.025;
      if (n === 0) return [];
      const maxEpics = n;
      let arr = [];
      for (let k = 1; k <= maxEpics; k++) {
        const prob = this.binomialProbability(n, k, pEpic);
        arr.push({
          k,
          odds: prob > 0 ? `1 in ${Math.round(1/prob).toLocaleString()}` : 'Extremely rare',
          prob
        });
      }
      return arr;
    }
  },
  mounted() {
    this.buyMods(10);
  }
}
</script>

<style scoped>
  .buy-buttons {
    margin-top: 30px;
    display: flex;
    justify-content: center;
    gap: 10px;
    flex-wrap: wrap;
  }

  .buy-button {
    padding: 16px 32px;
    border-radius: 5px;
    border: 1px solid #ccc;
    background-color: #343434;
    cursor: pointer;
    font-size: 1.2rem;
    width: 100%;
    touch-action: manipulation;
    user-select: none;
  }

  /* Prevent double-tap zoom on iOS */
  @media (hover: none) and (pointer: coarse) {
    .buy-button {
      touch-action: manipulation;
    }
  }

  .results {
    margin-top: 20px;
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #ccc;
    background-color: #343434;
    display: flex;
    gap: 5px;
    justify-content: center;
  }

  .result-container {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    border: 1px solid #ccc;
    overflow: hidden;
  }

  .icon {
    width: 100%;
    height: 100%;
  }

  .common {
    background-color: #686868;
  }

  .rare {
    background-color: hsl(196, 84%, 53%);
  }

  .epic {
    background-color: #ff03dd;
  }

  hr {
    margin: 20px 0;
  }

</style>
