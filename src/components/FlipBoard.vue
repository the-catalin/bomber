<template>
  <div class="container">
    <div class v-if="!teams.length">
      <div class="info">
        <a href="https://asq.ro">
          <img class="center" src="../assets/header-logo.png" alt="asq" />
        </a>
        <p class="tac">prezintă</p>

        <img
          class="flipper center"
          src="../assets/flipper.jpg"
          alt="flipper logo"
        />

        <h2>Descriere</h2>

        <p>
          Un joc interactiv de testare a cunoștințelor ce poate fi utilizat la
          școală de către întreaga clasă, la orice materie și la orice nivel.
        </p>

        <p>
          Ai nevoie de un ecran pe care sa-l poată vedea toată clasa
          (videoproiector, tablă interactivă, etc).
        </p>

        <h2>Regulile jocului</h2>

        <ol>
          <li>Sala de clasă se împarte în două, trei sau patru echipe.</li>
          <li>
            Profesorul pregătește un număr de minim cinci întrebări pentru
            fiecare echipă.
          </li>
          <li>
            La întrebări răspund pe rând câte un elev din fiecare echipă,
            profesorul fiind cel care va numi elevul ce urmează să răspundă.
          </li>
          <li>
            Elevul are voie să aleagă un pătrățel de pe panou doar dacă răspunde
            corect.
          </li>
          <li>
            Atenție! Înainte ca elevul să întoarcă un pătrățel, trebuie să se
            asigure că echipa sa este selectată.
          </li>
          <li>
            Pentru fiecare pătrățel descoperit, o echipă poate câștiga un
            punctaj de la 1 la 95, sau poate descoperi una dintre cele cinci
            bombe existente pe panou. Bomba face ca punctajul echipei să se
            reseteze de la zero.
          </li>
          <li>
            Echipa câștigătoare este echipa care obține cel mai mare punctaj.
          </li>
        </ol>

        <h2>Începe jocul!</h2>
        <p>Alege numărul de echipe:</p>

        <p>
          <button
            class="button"
            v-for="nr of [2, 3, 4]"
            @click="chooseNr(nr)"
            :key="nr"
          >
            {{ nr }}
          </button>
        </p>
      </div>

      <div class="choose"></div>
    </div>

    <div class="flipboard" v-if="teams.length">
      <div class="square">
        <div class="flex">
          <div class="scene" v-for="card in cards" :key="card.id">
            <div
              class="card"
              v-on:click="flip(card)"
              :class="[card.isflipped, card.color, card.bomb]"
            >
              <div class="cardface cardface-front">{{ card.front }}</div>
              <div class="cardface cardface-back">
                <span class="backtext">{{ card.back }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="stats">
        <div
          v-for="team in teams"
          @click="selectTeam(team.id)"
          :class="[team.color, team.selected]"
          :key="team.id"
        >
          <p class="team">
            <span>{{ team.name }}</span>
            <span class="points">{{ team.points }}</span>
          </p>
        </div>
        <div>
          <button class="reset" @click="undo" v-show="undoStack.length">
            Anulează
          </button>
        </div>
        <div>
          <button class="reset" @click="reset">Resetează jocul</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  props: {},
  data() {
    return {
      cards: [],
      numbers: [],
      teams: [],
      allTeams: [
        {
          id: 1,
          name: 'Echipa albastră',
          color: 'blue',
          points: 0,
          selected: false,
        },
        {
          id: 2,
          name: 'Echipa roșie',
          color: 'red',
          points: 0,
          selected: false,
        },
        {
          id: 3,
          name: 'Echipa portocalie',
          color: 'yellow',
          points: 0,
          selected: false,
        },
        {
          id: 4,
          name: 'Echipa verde',
          color: 'green',
          points: 0,
          selected: false,
        },
      ],
      currentTeam: 1,
      letter: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'J'],
      undoStack: [],
      isflipping: false,
    };
  },

  methods: {
    chooseNr(nr) {
      this.teams = this.allTeams.slice(0, nr);

      this.selectTeam(1);
    },

    flip(card) {
      if (!this.isflipping && !card.isflipped) {
        card.isflipped = 'isflipped';
        card.color = this.teams[this.currentTeam - 1].color;

        const rand = Math.floor(Math.random() * this.numbers.length);
        card.back = this.numbers[rand];
        this.numbers.splice(rand, 1);

        const undoEl = {
          card: card,
          team: this.currentTeam,
        };

        if (card.back == 'bomb') {
          card.bomb = 'bomb';
          undoEl.lastScore = this.teams[this.currentTeam - 1].points;
          this.teams[this.currentTeam - 1].points = 0;
        } else {
          this.teams[this.currentTeam - 1].points += card.back;
        }

        this.undoStack.push(undoEl);

        this.currentTeam++;
        if (this.currentTeam > this.teams.length) {
          this.currentTeam = 1;
        }

        this.isflipping = true;
        setTimeout(() => {
          this.selectTeam(this.currentTeam);
          this.isflipping = false;
        }, 1000);
      }
    },

    selectTeam(id) {
      for (let i = 0; i < this.teams.length; i++) {
        this.teams[i].selected = '';
      }
      this.teams[id - 1].selected = 'selected';
      this.currentTeam = id;
    },

    undo() {
      if (
        !this.isflipping &&
        confirm('Ești sigur că vrei să anulezi ultima acțiune?')
      ) {
        const last = this.undoStack.splice(this.undoStack.length - 1, 1);
        last[0].card.isflipped = false;
        if (!isNaN(last[0].card.back)) {
          this.teams[last[0].team - 1].points -= last[0].card.back;
        } else {
          this.teams[last[0].team - 1].points = last[0].lastScore;
        }
        this.numbers.push(last[0].card.back);

        this.currentTeam = last[0].team;
        this.currentTeam++;
        if (this.currentTeam > this.teams.length) {
          this.currentTeam = 1;
        }
        this.selectTeam(this.currentTeam);

        this.isflipping = true;
        setTimeout(() => {
          last[0].card.bomb = false;
          last[0].card.color = 'lightgray';
          this.isflipping = false;
        }, 1000);
      }
    },

    reset() {
      if (confirm('Ești sigur că vrei să resetezi jocul?')) {
        location = location;
      }
    },
  },

  created() {
    for (let i = 1; i <= 95; i++) {
      this.numbers.push(i);
    }
    for (let i = 1; i <= 5; i++) {
      this.numbers.push('bomb');
    }

    for (let i = 1; i <= 100; i++) {
      this.cards.push({
        id: i,
        front: this.letter[Math.floor((i - 1) / 10)] + (i % 10 ? i % 10 : 10),
        bomb: false,
        isflipped: false,
        color: 'lightgray',
      });

      // console.log(Math.floor((i-1)/10), i%10);
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  user-select: none;
}
.info {
  padding: 20px;
  max-width: 800px;
  font-size: 20px;
}
.center {
  display: block;
  margin: auto;
}
.tac {
  text-align: center;
  margin: 0;
  margin-top: 10px;
  font-size: 16px;
}
.flipper {
  max-width: 350px;
  width: 100%;
  background-image: url('../assets/flipper.jpg');
  background-size: cover;
}
.choose {
  font-size: 30px;
}
.choose p {
  text-align: center;
}
.flipboard {
  font-family: Lato, Tahoma;
  overflow: hidden;
}

.stats {
  font-family: Roboto, Tahoma;
  padding-left: 60px;
  float: left;
  margin-top: 50px;
}

.square {
  max-width: 700px;
  height: 700px;
  float: left;
}

.flex {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
}
.flex .scene {
  flex: 1 1 auto;
  width: 10%;
  box-sizing: border-box;
  /*border:1px solid black;*/
  text-align: center;
}

.scene {
  border: 1px solid #ddd;
  perspective: 600px;
}

.card {
  position: relative;
  width: 100%;
  min-width: 30px;
  height: 100%;
  cursor: pointer;
  transform-style: preserve-3d;
  transform-origin: center right;
  transition: transform 1s;
}

.card.isflipped {
  transform: translateX(-100%) rotateY(-180deg);
}

.cardface {
  position: absolute;
  width: 100%;
  height: 100%;
  text-align: center;
  font-weight: bold;
  backface-visibility: hidden;
}

.cardface-front {
  background: #a5b5b6;
  color: #333;
}

.cardface-back {
  color: #eee;
  transform: rotateY(180deg);
}

.blue .team,
.blue .cardface-back {
  background-color: #3498db;
}
.red .team,
.red .cardface-back {
  background-color: #e74c3c;
}
.yellow .team,
.yellow .cardface-back {
  background-color: #f39c12;
}
.green .team,
.green .cardface-back {
  background-color: #27ae60;
}

.bomb .cardface-back {
  background-image: url('../assets/bomb.jpg');
  background-size: cover;
}
.bomb .backtext {
  display: none;
}

.team {
  display: flex;
  justify-content: space-between;
  cursor: pointer;
  padding: 10px;
  color: white;
  text-transform: uppercase;
}
.points {
  display: inline-block;
  min-width: 40px;
  text-align: right;
}
.selected::before {
  content: url('../assets/arrow.png');
  position: absolute;
  margin-left: -40px;
}

.reset {
  width: 100%;
  font-size: 16px;
}
.button {
  height: 60px;
  font-size: 22px;
  margin-right: 10px;
  margin-bottom: 10px;
}

.reset:focus,
.button:focus {
  outline: 0;
}
.reset,
.button {
  cursor: pointer;
  display: inline-block;
  padding: 0.5em 1em;
  border: 0.16em solid #333;
  margin: 0 0.3em 0.3em 0;
  box-sizing: border-box;
  text-decoration: none;
  text-transform: uppercase;
  font-family: 'Roboto', sans-serif;
  font-weight: 400;
  color: #333;
  background: white;
  text-align: center;
  transition: all 0.15s;
}
.reset:hover,
.button:hover {
  color: #000;
  border-color: #000;
}

.reset:active,
.button:active {
  color: #bbbbbb;
  border-color: #bbbbbb;
}

.reset {
  margin-top: 60px;
}

@media all and (max-width: 30em) {
  /*.reset{display:block;margin:0.4em auto;}*/
}

@media only screen and (max-width: 549px) {
  .square {
    max-width: 400px;
    height: 400px;
  }
  .cardface {
    line-height: 40px;
    font-size: 14px;
  }
  .team {
    font-size: 14px;
  }
  .selected::before {
    margin-top: 2px;
  }
  .reset {
    margin-top: 10px;
  }
  .stats {
    margin-top: 20px;
  }
}

@media only screen and (min-width: 550px) and (max-width: 709px) {
  .square {
    max-width: 300px;
    height: 300px;
  }
  .cardface {
    line-height: 30px;
    font-size: 14px;
  }
  .team {
    font-size: 13px;
  }
  .selected::before {
    margin-top: 2px;
  }
  .reset {
    margin-top: 0px;
  }
  .stats {
    margin-top: 0px;
  }
}

@media only screen and (min-width: 710px) and (max-width: 874px) {
  .square {
    max-width: 430px;
    height: 430px;
  }
  .cardface {
    line-height: 43px;
    font-size: 16px;
  }
  .team {
    font-size: 16px;
  }
  .selected::before {
    margin-top: 4px;
  }
  .reset {
    margin-top: 20px;
  }
}

@media only screen and (min-width: 875px) and (max-width: 991px) {
  .square {
    max-width: 550px;
    height: 550px;
  }
  .cardface {
    line-height: 55px;
    font-size: 20px;
  }
  .team {
    font-size: 18px;
  }
  .selected::before {
    margin-top: 4px;
  }
  .reset {
    margin-top: 40px;
  }
}

@media only screen and (min-width: 992px) and (max-width: 1199px) {
  .square {
    max-width: 650px;
    height: 650px;
  }
  .cardface {
    line-height: 65px;
    font-size: 24px;
  }
  .team {
    font-size: 20px;
  }
  .points {
    min-width: 60px;
  }
  .selected::before {
    margin-top: 7px;
  }
}

@media only screen and (min-width: 1200px) {
  .square {
    max-width: 800px;
    height: 800px;
  }
  .cardface {
    line-height: 80px;
    font-size: 28px;
  }
  .team {
    padding: 10px;
    font-size: 24px;
  }
  .points {
    min-width: 70px;
  }
  .selected::before {
    margin-top: 10px;
  }
}
</style>
