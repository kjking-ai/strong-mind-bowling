<template>
  <div>
    <h1>Bowling game</h1>
    <div v-if="score.length < 10">
      <button @click="checkPoint()">Click to simulate the shot</button>
    </div>
    <div v-if="score.length >= 10">
      <h3>GAME FINISH</h3>
    </div>

    <div style="margin-top: 20px" v-if="shot.firstShot === -1">
      <div v-for="index in 10" class="bowling" :key="index"></div>
    </div>
    <div style="margin-top: 20px" v-if="shot.firstShot !== -1">
      <div v-for="index in (10 - shot.firstShot)" class="bowling" :key="index"></div>
    </div>

    <div style="margin-top: 50px">
      <p>Round: <span style="color: orange">{{ round }}</span></p>
      <p>
        This shot:<br>
        <span v-if="shot.firstShot !== -1">First shot: {{ shot.firstShot }}</span><br>
        <span v-if="shot.secondShot !== -1">Secondo shot: {{ shot.secondShot }}</span>
      </p>

      <table>
        <tr>
          <th>Round</th>
          <th>First shot</th>
          <th>Second Shot</th>
          <th>Third Shot</th>
          <th>Partial Sum</th>
        </tr>
        <tr v-for="(point, index) in score" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ point.firstShot === 10 ? 'X' : point.firstShot }}</td>
          <td>{{ point.firstShot === 10 ? '' : point.secondShot }}</td>
          <td>{{ point.thirdShot === -1 ? '/' : point.thirdShot }}</td>
          <td>{{ point.partialSum }}</td>
        </tr>
      </table>

      <div style="margin-top: 25px">
        Global score: <span class="globalscore">{{ globalScore }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

interface Shots {
  firstShot: number;
  secondShot: number;
  thirdShot?: number;
  partialSum: number;
}
@Component
export default class Bowling extends Vue {
  // Global score
  public score: Shots[] = [];
  // Round
  public round: number = 0;
  // Shot 1 or 2
  public shot: Shots = {
    firstShot: -1,
    secondShot: -1,
    thirdShot: -1,
    partialSum: 0,
  };
  // Other variables
  public spare: boolean = false;
  public strike: boolean = false;

  public created() {
    console.log('Start game!');
  }

  public checkPoint() {
    let point = 0;
    if (this.shot.firstShot === -1) {
      // If it's the first shot, the max = 10
      point = Math.floor(Math.random() * (10 - 0 + 1)) + 0;
    } else {
      // If it's the second shot, the max = (10 - firstShot)
      point = Math.floor(Math.random() * ((10 - this.shot.firstShot) - 0 + 1)) + 0;
    }

    console.log('Shot: ' + point);

    // Not a strike...
    if (point !== 10) {
      // Check if it's the first or the second shot
      if (this.shot.firstShot === -1) {
        this.shot.firstShot = point;
        this.shot.partialSum += point;

        // If before I take a SPARE, now add the point of the first shot
        if (this.spare) {
          this.score[this.round - 1].partialSum += point;
          this.spare = false;
        }

        // If before I take a STRIKE, now add the point of the first shot
        if (this.strike) {
          this.score[this.round - 1].partialSum += point;
        }
      } else {
        this.shot.secondShot = point;
        this.shot.partialSum += point;
        this.score.push(this.shot);

        // I TAKE A SPARE!
        if ((this.shot.firstShot + this.shot.secondShot) === 10) {
          this.spare = true;
        }

        // If before I take a STRIKE, now add the point of the first shot
        if (this.strike) {
          this.score[this.round - 1].partialSum += point;
          this.strike = false;
        }

        this.round ++;
        this.resetShot();
      }
    } else {
      // If before I take a SPARE, now add the point of the first shot
      if (this.spare) {
        this.score[this.round - 1].partialSum += point;
        this.spare = false;
      }

      // If before I take a STRIKE, now add the point of the first shot
      if (this.strike) {
        this.score[this.round - 1].partialSum += point;
        this.strike = false;
      }

      // STRIKE!
      this.strike = true;
      this.shot.firstShot = point;
      this.shot.secondShot = 0;
      this.shot.partialSum += point;

      this.score.push(this.shot);

      this.round ++;
      this.resetShot();
    }
  }

  public resetShot() {
    this.shot = {
      firstShot: -1,
      secondShot: -1,
      thirdShot: -1,
      partialSum: 0
    };
  }

  public get globalScore() {
    let sum = 0;
    for (let i = 0; i < this.score.length; i ++) {
      sum += this.score[i].partialSum;
    }
    return sum;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  button {
    margin-right: 10px;
    height: 30px;
    width: 150px;
  }

  .bowling {
    height: 50px;
    width: 50px;
    border-radius: 25px;
    border: 1px solid black;
    display: inline-block;
    margin-right: 10px;
  }

  .globalscore {
    color: green;
    font-size: 18px;
    font-weight: bold;
  }

  table {
    font-family: arial, sans-serif;
    border-collapse: collapse;
    width: 50%;
  }

  td, th {
    border: 1px solid #dddddd;
    text-align: left;
    padding: 8px;
  }

  tr:nth-child(even) {
    background-color: #dddddd;
  }
</style>
