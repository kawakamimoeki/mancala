<template>
<p class="player">2P</p>
<div class="board">
  <div class="goal john" id="13" :class="{ planted: isPlanted(6) }">
    {{ pockets[13] }}
  </div>
  <div class="pockets">
    <div class="pockets--reverse">
    <button v-for="index in owned['john']"
          v-on:click="plant"
          :id="index"
          :key="index"
          class="pocket john"
          :class="{ goal: isGoal(index), planted: isPlanted(index) }"
          :disabled="!plantable(index)">
      {{ pockets[index] }}
    </button>
    </div>
    <div>
      <button v-for="index in owned['jane']"
            v-on:click="plant"
            :id="index"
            :key="index"
            class="pocket jane"
            :class="{ goal: isGoal(index), planted: isPlanted(index) }"
            :disabled="!plantable(index)">
        {{ pockets[index] }}
      </button>
    </div>
  </div>
  <div class="goal jane" id="6" :class="{ planted: isPlanted(6) }">
    {{ pockets[6] }}
  </div>
</div>
<p class="player player--1">1P</p>
</template>

<style scoped>
.palyer {
  margin: 10px;
}
.player--1 {
  text-align: right;
  margin: 10px;
}
.board {
  display: flex;
  width: 100%;
  flex-wrap: wrap;
  height: 200px;
  box-sizing: border-box;
}
.goal {
  border: solid 1px gray;
  height: 180px;
  width: calc(100% / 8 - 20px);
  box-sizing: border-box;
  line-height: 200px;
  text-align: center;
  margin: 10px;
}
.john {
  background: #def4ff;
  border: 1px #00aaff solid;
}
.jane {
  background: #ffe6f7;
  border: 1px #ff8ada solid;
}
.pockets {
  height: 200px;
  width: calc(100% * 3 / 4);
  box-sizing: border-box;
}
.pockets--reverse {
  display: flex;
  flex-direction: row-reverse;
}
.pocket {
  width: calc(100% / 6 - 20px);
  box-sizing: border-box;
  height: 80px;
  margin: 10px;
}
.pocket:hover {
  border-width: 2px;
}
.pocket:disabled {
  border-width: 1px;
}
.planted {
  border-width: 2px;
}
</style>

<script>
export default {
  data() {
    return {
      frame: 500,
      turn: 'jane',
      opponent: 'john',
      planted: null,
      goals: { jane: 6, john: 13 },
      owned: { jane: [0, 1, 2, 3, 4, 5], john: [7, 8, 9, 10, 11, 12] },
      seed: 4,
    }
  },
  created: function () {
    const farm = []
    for (let i = 0; i < 6; i++) farm.push(this.seed)
    this.pockets = [...farm, 0, ...farm, 0]
  },
  methods: {
    plantable: function (index) {
      return this.owned[this.turn].includes(index)
    },
    plant: async function (event) {
      const origin = event.target.id
      var seeds = this.pockets[origin]
      this.pockets[origin] = 0
      var target = parseInt(origin)
      for (var i = 1; i <= seeds; i++) {
        target += 1
        if (target > 13) target = 14 - target
        if (!this.isOpponentGoal(target)) {
          this.planted = target
          this.pockets[target] += 1
          await this.sleep(this.frame)
        } else {
          seeds += 1
        }
      }
      this.planted = null
      if(this.isEmpty()) {
        await this.finish()
      }
      if (target === this.goals[this.turn]) {
        window.alert('ぴったりゴール！')
      } else {
        if (this.pockets[target] === 1) {
          await this.steal(target)
        }
        this.changeTurn()
      }
    },
    finish: async function () {
      (async () => {
          for (let pocket of this.owned[this.opponent]) {
            this.planted = pocket
            const seeds = this.pockets[pocket]
            await this.sleep(this.frame)
            this.pockets[this.goals[this.opponent]] += seeds
            this.pockets[pocket] = 0
          }
          this.planted = null

          await this.sleep(2000)

          if (this.goals['jane'] > this.goals['john']) {
            window.alert('1P Win!')
          } else if (this.goals['john'] > this.goals['jane']) {
            window.alert('2P Win!')
          } else {
            window.alert('引き分け!')
          }

          this.pockets = []
      })()
    },
    opposite (index) {
      if (this.isGoal(index)) {
        return null
      } else {
        return 12 - index
      }
    },
    steal: async function (index) {
      await this.sleep(this.frame)
      window.alert('横取り！')
      const oppositeSeeds = this.pockets[this.opposite(index)]
      this.pockets[this.goals[this.turn]] += (oppositeSeeds + 1)
      this.pockets[this.opposite(index)] = 0
      this.pockets[index] = 0
    },
    changeTurn () {
      const turn = this.turn
      this.turn = this.opponent
      this.opponent = turn
    },
    isGoal: function (index) {
      return Object.values(this.goals).includes(index)
    },
    isEmpty () {
      const notEmpties = this.owned[this.turn].find((pocket) => {
        return this.pockets[pocket] > 0
      })
      return notEmpties === undefined
    },
    isPlanted (index) {
      return index === this.planted
    },
    isOpponentGoal (index) {
      return this.goals[this.opponent] === index
    },
    sleep: function (ms) {
      return new Promise((resolve) => {
        setTimeout(resolve, ms)
      })
    }
  }
}
</script>
