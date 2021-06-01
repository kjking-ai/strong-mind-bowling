<template>
  <div class="wrapper">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <body>
      <div class="navbar-bowling">
        <b-navbar type="light" variant="light">
          <b-navbar-nav>
            <img class="bowling-img" src="../assets/bowling-pin.png" alt="" />
            <b-nav-item href="#">Bowling Score Tracker</b-nav-item>
          </b-navbar-nav>
        </b-navbar>
      </div>
    
      <div class="bowling-container">
        <div v-if="round <= tracker">
          <input
            ref="afterClick"
            type="number"
            placeholder="Enter Value (0-10)"
            id="userBowlingValue"
            class="input"
            v-model="point"
          />
          <br />
          <button @click="checkPoint()">Enter Pins</button> <br />
        </div>
        <b-button v-b-modal.modal-multi-1>Help</b-button>
        <b-modal id="modal-multi-1" size="lg" title="Help" ok-only no-stacking>
          <p class="my-2">
            - Enter the amount of pins knocked down for each roll, Then select
            enter pins! <br />
            - Each value entered is equal to one roll. <br />
            - Value entered must be between 0 and 10.
          </p>
        </b-modal>
        <div v-if="round > tracker">
          <h3>GAME FINISH</h3>
        </div>
        <button class="restartBtn" @click="restartGame()">Restart</button>
        <br />

        <div>
          <p>
            <span v-if="shot.firstShot !== -1 && this.round <= tracker"
              >First shot: {{ shot.firstShot }}</span
            ><br />
          </p>

          <div class="globalscore-title" style="margin-top: 10px">
            Global score: <span class="globalscore">{{ globalScore }}</span>
          </div>

          <table>
            <tr>
              <th>Frame</th>
              <th>Shot 1</th>
              <th>Shot 2</th>
              <th>Shot 3</th>
              <th>Total</th>
            </tr>
            <tr v-for="(point, index) in score" :key="index">
              <td>{{ index + 1 }}</td>
              <td>{{ point.firstShot === 10 ? "X" : point.firstShot }}</td>
              <td>
                {{
                  point.firstShot === 10 && point.secondShot === 0
                    ? ""
                    : point.secondShot
                }}
              </td>
              <td>{{ point.thirdShot === -1 ? "/" : point.thirdShot }}</td>
              <td>{{ point.total }}</td>
            </tr>
          </table>
        </div>
      </div>
      <footer class="bowling-footer"></footer>
    </body>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

interface Shots {
  firstShot: number;
  secondShot: number;
  thirdShot: number;
  partialSum: number;
  total: number;
}

@Component
export default class Bowling extends Vue {
  //Current Point
  public point = "";
  // Global Score
  public score: Shots[] = [];
  // Round
  public round: number = 1;
  // Global Tracker
  public tracker: number = 10;
  // Used for 10th Framer to Track Strikes
  public strikeTracker: number = 0;
  // Shot 1 or 2
  public shot: Shots = {
    firstShot: -1,
    secondShot: -1,
    thirdShot: -1,
    partialSum: 0,
    total: 0,
  };
  // Other variables
  public spare: boolean = false;
  public strike: boolean = false;
  public thirdShot: boolean = false;

  $refs!: {
    afterClick: HTMLFormElement;
  };

  public created() {
    console.log("Start game!");
  }
  // need to figure out how to get input here
  public checkPoint() {
    if (this.point !== "") {
      let point: number = +this.point;
      point = Math.floor(point);
      this.point = "";
      this.$refs.afterClick.focus();
      if (point < 0 || point > 10) {
        console.log("error");
      } else {
        // NOT a strike...
        if (point !== 10) {
          //FIRST SHOT
          if (this.shot.firstShot === -1) {
            // Add First Shot to Partial
            if (this.round <= 10) {
              this.shot.firstShot = point;
              console.log("error1");
              this.shot.partialSum += point;
              console.log("error2");
              if (this.strike === true) {
                this.score[this.round - 2].partialSum += point;
                this.score[this.round - 2].total += point;
                if (
                  this.round >= 3 &&
                  this.score[this.round - 3].firstShot == 10
                ) {
                  this.score[this.round - 3].partialSum += point;
                  this.score[this.round - 3].total += point;
                }
              }
              if (this.spare === true) {
                this.score[this.round - 2].partialSum += point;
                this.score[this.round - 2].total += point;
                this.spare = false;
              }
            } else {
              if (this.strike === true) {
                console.log("error1");
                this.score[this.round - 2].partialSum += point;
                this.score[this.round - 2].total += point;
                if (
                  this.round >= 3 &&
                  this.score[this.round - 3].firstShot == 10
                ) {
                  this.score[this.round - 3].partialSum += point;
                  this.score[this.round - 3].total += point;
                  this.score[this.score.length - 1].secondShot = point;
                  this.strike = false;
                }
              } else {
                this.score[this.score.length - 1].thirdShot = point;
                this.score[this.score.length - 1].total += point;
                this.score[this.score.length - 1].partialSum += point;
                this.round = 46;
              }
            }
          } else {
            //SECOND SHOT
            if (point + this.shot.firstShot > 10) {
              console.log("error");
            } else {
              if (this.shot.secondShot == 0) {
                this.shot.secondShot = 0;
              } else {
                this.shot.secondShot = point;
              }

              if (this.strike === true) {
                this.score[this.round - 2].partialSum += point;
                this.score[this.round - 2].total += point;
                this.strike = false;
              }
              // CHECKING & SETTING SPARE
              if (this.shot.firstShot + this.shot.secondShot === 10) {
                console.log("THIS IS A SPARE");
                this.spare = true;
                if (this.round == 10) {
                  this.tracker = 11;
                }
              }

              if (this.tracker <= 11) {
                this.shot.partialSum += point;
                this.score.push(this.shot);
                this.round++;
                this.resetShot();
              } else {
                this.shot.partialSum += point;
                console.log("round: ", this.round);
                console.log("game over: ", point);
                this.round++;
              }
            }
          }
        } else if (this.shot.firstShot == -1) {
          //STRIKE!!!
          this.shot.firstShot = point;
          this.shot.secondShot = 0;
          this.shot.partialSum = point;

          if (this.spare === true) {
            this.score[this.round - 2].partialSum += point;
            this.score[this.round - 2].total += point;
            this.spare = false;
          }

          if (this.strike === true) {
            // error happens here
            if (this.tracker < 12) {
              if (this.score[this.round - 2].firstShot == 10) {
                this.score[this.round - 2].partialSum += point;
                this.score[this.round - 2].total += point;
                if (
                  this.round >= 3 &&
                  this.score[this.round - 3].firstShot == 10
                ) {
                  this.score[this.round - 3].partialSum += point;
                  this.score[this.round - 2].total += point;
                  this.score[this.round - 3].total += point;
                }
              }
            } else {
              this.score[this.score.length - 1].partialSum += point;
              this.score[this.score.length - 1].total += point;
              this.score[this.score.length - 1].secondShot = 10;
              this.score[this.score.length - 1].thirdShot = 10;
            }
          }

          if (this.round == 10) {
            this.tracker = 11;
          }

          if (this.round == 11) {
            this.tracker = 12;
          }
          console.log("Tracker: ", this.tracker);
          if (this.round <= 10) {
            this.score.push(this.shot);
          }
          this.round++;
          this.resetShot();
          this.strike = true;
          console.log("Round: ", this.round);
        }
      }
    }
    if (this.round > 1 && this.round < 13) {
      console.log("Round-2: ", this.round - 2);
      this.score[this.round - 2].total = this.globalScore;
    } else {
    }
  }

  public resetShot() {
    this.shot = {
      firstShot: -1,
      secondShot: -1,
      thirdShot: -1,
      partialSum: 0,
      total: 0,
    };
  }

  public restartGame() {
    window.location.reload();
  }

  public get globalScore() {
    let sum = 0;
    for (let i = 0; i < this.score.length; i++) {
      sum += this.score[i].partialSum;
    }
    return sum;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped >
Button {
  box-shadow: inset 0px 1px 0px 0px #cf866c;
  background: linear-gradient(to bottom, #d0451b 5%, #bc3315 100%);
  background-color: #d0451b;
  border-radius: 3px;
  border: 1px solid #942911;
  display: inline-block;
  cursor: pointer;
  color: #ffffff;
  font-family: Arial;
  font-size: 0.7em;
  padding: 8px 24px;
  text-decoration: none;
  text-shadow: 0px 1px 0px #854629;
  width: 100%;
  max-width: 200px;
  margin-left: 1%;
  margin-right: auto;
  margin-left: auto;
  margin-bottom: 1%;
}
Button:hover {
  background: linear-gradient(to bottom, #f24437 5%, #c62d1f 100%);
  background-color: #f24437;
}
Button:active {
  position: relative;
  top: 1px;
}

body {
  width: 100%;
  height: 100%;
}
.bowling-footer {
  background-color: #d0451b;
  width: 80%;
  height: 5%;
  margin-right: auto;
  margin-left: auto;
}
.bowling-img {
  display: block;
  float: left;
  width: 10%;
  height: 4%;
  margin-top: -2%;
}
.bowling-title {
  display: block;
  width: 40%;
  height: 20%;
}
.bowling-container {
  margin-right: auto;
  margin-left: auto;
  width: 80%;
  top: 0;
  bottom: 0;
  padding: 40px;
  border-radius: 15px 50px;
}

.bowling {
  height: 50px;
  width: 50px;
  border-radius: 25px;
  border: 1px solid black;
  display: inline-block;
  margin-right: 10px;
}

.globalscore-title {
  font-size: 2vw;
  font-weight: bold;
}
.globalscore {
  color: red;
  font-size: 2vw;
  font-weight: bold;
}

.input {
  padding: 5px;
  font-size: 0.8em;
  border-width: 1px;
  border-color: #cccccc;
  background-color: #ffffff;
  color: #000000;
  border-style: solid;
  border-radius: 0px;
  box-shadow: 0px 0px 5px rgba(66, 66, 66, 0.75);
  text-shadow: 0px 0px 5px rgba(66, 66, 66, 0.75);
  width: 100%;
  max-width: 200px;
  margin-bottom: 1%;
}
.input:focus {
  outline: none;
}

table {
  margin-left: auto;
  margin-right: auto;
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
  height: 100%;
  table-layout: fixed;
}

td,
th {
  border: 1px solid #dddddd;
  padding: 8px;
  width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  word-wrap: break-word;
  font-size: 2vw;
}

tr:nth-child(even) {
  background-color: #dddddd;
}

.navbar-bowling {
  width: 80%;
  margin-left: auto;
  margin-right: auto;
  background-color: #e3f2fd;
  font-family: Impact, Charcoal, sans-serif;
  font-size: 3vw;
  letter-spacing: 2px;
  word-spacing: 2px;
  color: #000000;
  font-weight: 700;
  text-decoration: overline solid rgb(68, 68, 68);
  font-style: normal;
  font-variant: normal;
  text-transform: uppercase;
}
@media (min-width: 540px) {
  .restartBtn {
    float: right;
  }
}

.wrapper {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  overflow: auto;
}
</style>
