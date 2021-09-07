<template>
  <v-app id="inspire" style="background-color: #FFFCFF;">
    <div class="text-center" id="loading">
      <v-progress-circular
        :size="200"
        color="primary"
        indeterminate
      ></v-progress-circular>
    </div>
    <v-app-bar flat app style="background-color: #BFEDEF;">
      <v-container>
        <v-row style="padding-top: 20px;">
          <v-col cols="10" sm="5" md="3">
            <v-autocomplete  :items="Object.keys(regionsValues)" v-model="region"></v-autocomplete>
          </v-col>
          <v-col cols="10" sm="6" md="3">
            <v-text-field id="searchBox"
              label="Enter your summoner name"
              v-model="summoner"
            ></v-text-field>
          </v-col>
          <v-col cols="2" sm="1">
            <v-icon
              size="40px"
              style="cursor: pointer"
              v-on:click="getSummoner(summoner)"
              >mdi-magnify</v-icon
            >
          </v-col>
        </v-row>
      </v-container>
    </v-app-bar>

    <v-main>
      <v-container id="main">
        <v-row>
          <v-col cols="12">
            <v-card
              elevation="1"
            >

            <span><h1>{{ summonerData.name }}</h1></span>
            <div style="height: 130px;
            padding: 10px;">
            <v-img
              contain
              max-height="100"
              max-width="100"
              style="overflow:visible;"
              :src="require('../src/assets/profileicon/' + summonerData.profileIconId + '.png')">
                <v-img
                  contain
                  max-height="120"
                  max-width="120"
                  style="
                  position:absolute;
                  top: 50%;
                  left: 50%;
                  transform:translate(-50%,-50%);

                  "
                  :src="require('../src/assets/leagueBorders/' + tier + '.png')"
                >
                </v-img>
              </v-img>
            </div>
            </v-card>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="2">
            <v-card align="center" style="background-color: #FFFFFF">
              <v-img
                contain
                max-height="150"
                max-width="250"
                id="leagueEmblem"
                :src="require('../src/assets/rankedEmblems/' + tier + '.png')"
              ></v-img>
              {{ tier[0] + tier.slice(1).toLowerCase() }} {{ rank }}
              <v-divider></v-divider>
              {{ lp }} LP| <span style="color: green">{{ wins }}</span> / <span style="color: red;">{{ losses }}</span>
              <br>
              Win Rating: {{ winRating }}%
            </v-card>
          </v-col>
          <v-col cols="12" md="8">
            <v-expansion-panels id="matchHistory">
              <v-expansion-panel
              v-for="(item,i) in matchList.length" :key="i"
              >
                <v-expansion-panel-header align="center" :style="myStyle[i]">
                  <v-row>
                    <v-col cols="6" sm="3" md="3" align="center">
                    {{ descriptions[i] }} <br>
                    {{ gameStarts[i] }} <br> <br>
                    {{ summonerScore[i].result }}
                    </v-col>
                    <v-divider vertical style="margin-left: 10px;"></v-divider>
                    <v-col cols="3" sm="1" style="padding: 0px 0px 0px 5px!important" align="center" :align-self="'center'">
                      <span>
                      <v-img class="rounded-circle"
                        contain
                        max-height="60"
                        max-width="60"
                        :src="require('../src/assets/champion/' + summonerScore[i].champName + '.png')"
                      ></v-img>
                      </span>
                    </v-col>
                    <v-col cols="2" sm="1" style="padding-left: 5px !important" aling="center" :align-self="'center'">
                          <span>
                        <v-img
                          contain
                          max-height="30"
                          max-width="30"
                          :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[summonerScore[i].summoner1])"
                        ></v-img>
                        <v-img
                          contain
                          max-height="30"
                          max-width="30"
                          :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[summonerScore[i].summoner2])"
                        ></v-img>
                        </span>
                      </v-col>
                    <v-col cols="6" sm="3" md="3" :align-self="'center'" align="center">
                      {{ summonerScore[i].champLevel }} <br>
                      {{ summonerScore[i].minions }} CS <br>
                      <span style="font-size:12px;">{{ Math.round(summonerScore[i].minions / gameDurations[i]) }} cs/min
                      </span>
                    </v-col>
                    <v-col cols="5" sm="3" md="3" style="margin-left: 10px;" :align-self="'center'" align="center" >
                      
                      <span style="color: #07bd04"> {{ summonerScore[i].kills }}</span> / 
                      <span style="color: #b50424">{{ summonerScore[i].deaths }}</span> / 
                      <span style="color: #0716a3">{{ summonerScore[i].assists }}</span> <br>
                      <span style="font-size: 12px;">{{ Math.round(((summonerScore[i].kills + summonerScore[i].assists) / summonerScore[i].deaths + Number.EPSILON) * 100) / 100 }} KDA</span><br>
                      {{ gameDurations[i] }} min
                    </v-col>
                  </v-row>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <v-simple-table class="big">
                    <template v-slot:default>
                      <thead>
                        <tr>
                          <th style="width: 16% !important;" class="text-center" >
                            {{teams[i][1].win}}
                          </th>
                          <th style="width: 16% !important; max-width: 16%" class="text-center">
                            Username
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            KDA
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            Damage
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            CS
                          </th>
                          <th style="width: 16% !important; max-width: 16%" class="text-center">
                            Items
                          </th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="player in matchPlayers[i].slice(0, 5)"
                            :key="player.championName">
                          <td align="center">
                            <v-img
                              style="display:inline-block !important;"
                              contain
                              max-height="40"
                              max-width="40"
                              :src="require('../src/assets/champion/' + player.championName.toLowerCase() + '.png')"
                            ></v-img>
                            <div style="display:inline-block !important;">
                              <v-img
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner1Id])"
                              ></v-img>
                              <v-img
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner2Id])"
                              ></v-img>
                            </div>
                          </td>
                          <td align="center">
                            <span class="link" v-on:click="getSummoner(player.summonerName)">{{ player.summonerName }}</span>
                          </td>
                          <td align="center">
                            <span style="color: #07bd04"> {{ player.kills }}</span>/<span style="color: #b50424">{{ player.deaths }}</span>/<span style="color: #0716a3">{{ player.assists }}</span> <br>
                            <span style="font-size: 8px;">{{ Math.round(((player.kills + player.assists) / player.deaths + Number.EPSILON) * 100) / 100 }} KDA</span>
                          </td>
                          <td align="center">
                            {{player.totalDamageDealtToChampions}}
                            <v-progress-linear :value="Math.round(player.totalDamageDealtToChampions / bigDmg[i].totalDmg * 100)"></v-progress-linear>
                          </td>
                          <td align="center">
                            {{player.totalMinionsKilled + player.neutralMinionsKilled}}
                          </td>
                          <td align="center">
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item0 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item1 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item2 + '.png')"
                              ></v-img>
                            </div>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item3 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item4 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item5 + '.png')"
                              ></v-img>
                            </div>
                          </td>
                        </tr>
                      </tbody>

                    </template>
                  </v-simple-table>
                  <v-simple-table class="big">
                    <template v-slot:default>
                      <thead>
                        <tr>
                        <th style="width: 16% !important;" class="text-center" >
                            {{teams[i][1].win}}
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            Username
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            KDA
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            Damage
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            CS
                          </th>
                          <th style="width: 16% !important;" class="text-center">
                            Items
                          </th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="player in matchPlayers[i].slice(5, 10)"
                            :key="player.championName">
                          <td align="center">
                            <v-img
                              style="display:inline-block !important;"
                              contain
                              max-height="40"
                              max-width="40"
                              :src="require('../src/assets/champion/' + player.championName.toLowerCase() + '.png')"
                            ></v-img>
                            <div style="display:inline-block !important;">
                              <v-img
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner1Id])"
                              ></v-img>
                              <v-img
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner2Id])"
                              ></v-img>
                            </div>
                          </td>
                          <td align="center">
                            <span class="link" v-on:click="getSummoner(player.summonerName)">{{ player.summonerName }}</span>
                          </td>
                          <td align="center">
                            <span style="color: #07bd04"> {{ player.kills }}</span>/<span style="color: #b50424">{{ player.deaths }}</span>/<span style="color: #0716a3">{{ player.assists }}</span> <br>
                            <span style="font-size: 8px;">{{ Math.round(((player.kills + player.assists) / player.deaths + Number.EPSILON) * 100) / 100 }} KDA</span>
                          </td>
                          <td align="center">
                            {{player.totalDamageDealtToChampions}}
                            <v-progress-linear :value="Math.round(player.totalDamageDealtToChampions / bigDmg[i].totalDmg * 100)"></v-progress-linear>
                          </td>
                          <td align="center">
                            {{player.totalMinionsKilled + player.neutralMinionsKilled}}
                          </td>
                          <td align="center">
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item0 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item1 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item2 + '.png')"
                              ></v-img>
                            </div>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item3 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item4 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/item/' + player.item5 + '.png')"
                              ></v-img>
                            </div>
                          </td>
                          
                        </tr>
                      </tbody>
                    </template>
                  </v-simple-table>
                  <v-simple-table class="small">
                    <template v-slot:default>
                      <thead>
                        <tr>
                          <th style="width: 100px !important;"  class="text-center" >
                            {{teams[i][1].win}}
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            Username
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            KDA
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            Damage
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            CS
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            Items
                          </th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="player in matchPlayers[i].slice(0, 5)"
                            :key="player.championName">
                          <td>
                            <v-img
                              style="display:inline-block !important;"
                              contain
                              max-height="40"
                              max-width="40"
                              :src="require('../src/assets/champion/' + player.championName.toLowerCase() + '.png')"
                            ></v-img>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner1Id])"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner2Id])"
                              ></v-img>
                            </div>
                          </td>
                          <td>
                            <span class="link" v-on:click="getSummoner(player.summonerName)">{{ player.summonerName }}</span>
                          </td>
                          <td style="font-size:12px">
                            <span style="color: #07bd04"> {{ player.kills }}</span>/<span style="color: #b50424">{{ player.deaths }}</span>/<span style="color: #0716a3">{{ player.assists }}</span> <br>
                            <span style="font-size: 8px;">{{ Math.round(((player.kills + player.assists) / player.deaths + Number.EPSILON) * 100) / 100 }} KDA</span>
                          </td>
                          <td>
                            {{player.totalDamageDealtToChampions}}
                            <v-progress-linear :value="Math.round(player.totalDamageDealtToChampions / bigDmg[i].totalDmg * 100)"></v-progress-linear>
                          </td>
                          <td>
                            {{player.totalMinionsKilled + player.neutralMinionsKilled}}
                          </td>
                          <td>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item0 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item1 + '.png')"
                              ></v-img>
                            </div>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item2 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item3 + '.png')"
                              ></v-img>
                            </div>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item4 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item5 + '.png')"
                              ></v-img>
                            </div>
                          </td>
                        </tr>
                      </tbody>

                    </template>
                  </v-simple-table>
                  <v-simple-table class="small">
                    <template v-slot:default>
                      <thead>
                        <tr>
                          <th style="width: 100px !important;"  class="text-center" >
                            {{teams[i][1].win}}
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            Username
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            KDA
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            Damage
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            CS
                          </th>
                          <th style="width: 100px !important;" class="text-center">
                            Items
                          </th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="player in matchPlayers[i].slice(5, 10)"
                            :key="player.championName">
                          <td>
                            <v-img
                              style="display:inline-block !important;"
                              contain
                              max-height="40"
                              max-width="40"
                              :src="require('../src/assets/champion/' + player.championName.toLowerCase() + '.png')"
                            ></v-img>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner1Id])"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="20"
                                max-width="20"
                                :src="require('../src/assets/summonerSpells/' + summonerSpellsIDs[player.summoner2Id])"
                              ></v-img>
                            </div>
                          </td>
                          <td>
                            <span class="link" v-on:click="getSummoner(player.summonerName)">{{ player.summonerName }}</span>
                          </td>
                          <td style="font-size:12px">
                            <span style="color: #07bd04"> {{ player.kills }}</span>/<span style="color: #b50424">{{ player.deaths }}</span>/<span style="color: #0716a3">{{ player.assists }}</span> <br>
                            <span style="font-size: 8px;">{{ Math.round(((player.kills + player.assists) / player.deaths + Number.EPSILON) * 100) / 100 }} KDA</span>
                          </td>
                          <td>
                            {{player.totalDamageDealtToChampions}}
                            <v-progress-linear :value="Math.round(player.totalDamageDealtToChampions / bigDmg[i].totalDmg * 100)"></v-progress-linear>
                          </td>
                          <td>
                            {{player.totalMinionsKilled + player.neutralMinionsKilled}}
                          </td>
                          <td>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item0 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item1 + '.png')"
                              ></v-img>
                            </div>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item2 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item3 + '.png')"
                              ></v-img>
                            </div>
                            <div>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item4 + '.png')"
                              ></v-img>
                              <v-img
                                style="display:inline-block !important;"
                                contain
                                max-height="15"
                                max-width="15"
                                :src="require('../src/assets/item/' + player.item5 + '.png')"
                              ></v-img>
                            </div>
                          </td>
                        </tr>
                      </tbody>
                    </template>
                  </v-simple-table>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<style scoped>



.v-progress-circular {
  margin: 1rem;
}

.link {
  cursor: pointer;
}

td {
  padding: 10px !important;
}

#inspire {
  background-image: url("../src/assets/background.jpg");
  
  background-repeat: no-repeat;
  
  background-attachment: fixed;
}
#loading {
  display: none;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
  z-index: 20000;
}

#main {
  display: none;
}

.small {
  display: none;
}

@media only screen and (max-width: 600px) {
  .small {
    display: block;
  }
  .big {
    display: none;
  }
}

</style>

<script>
export default {
  name: "App",

  data: () => ({

    riotkey: "",

    metaInfo: {
      title: 'Hello'
    },

    queIDs: [
      {
          "queueId": 0,
          "map": "Custom ",
          "description": null,
          "notes": null
      },
      {
          "queueId": 72,
          "map": "Howling Abyss",
          "description": "1v1 Snowdown Showdown ",
          "notes": null
      },
      {
          "queueId": 73,
          "map": "Howling Abyss",
          "description": "2v2 Snowdown Showdown ",
          "notes": null
      },
      {
          "queueId": 75,
          "map": "Summoner's Rift",
          "description": "6v6 Hexakill ",
          "notes": null
      },
      {
          "queueId": 76,
          "map": "Summoner's Rift",
          "description": "Ultra Rapid Fire ",
          "notes": null
      },
      {
          "queueId": 78,
          "map": "Howling Abyss",
          "description": "One For All: Mirror Mode ",
          "notes": null
      },
      {
          "queueId": 83,
          "map": "Summoner's Rift",
          "description": "Co-op vs AI Ultra Rapid Fire ",
          "notes": null
      },
      {
          "queueId": 98,
          "map": "Twisted Treeline",
          "description": "6v6 Hexakill ",
          "notes": null
      },
      {
          "queueId": 100,
          "map": "Butcher's Bridge",
          "description": "5v5 ARAM ",
          "notes": null
      },
      {
          "queueId": 310,
          "map": "Summoner's Rift",
          "description": "Nemesis ",
          "notes": null
      },
      {
          "queueId": 313,
          "map": "Summoner's Rift",
          "description": "Black Market Brawlers ",
          "notes": null
      },
      {
          "queueId": 317,
          "map": "Crystal Scar",
          "description": "Definitely Not Dominion ",
          "notes": null
      },
      {
          "queueId": 325,
          "map": "Summoner's Rift",
          "description": "All Random ",
          "notes": null
      },
      {
          "queueId": 400,
          "map": "Summoner's Rift",
          "description": "5v5 Draft Pick ",
          "notes": null
      },
      {
          "queueId": 420,
          "map": "Summoner's Rift",
          "description": "5v5 Ranked Solo ",
          "notes": null
      },
      {
          "queueId": 430,
          "map": "Summoner's Rift",
          "description": "5v5 Blind Pick ",
          "notes": null
      },
      {
          "queueId": 440,
          "map": "Summoner's Rift",
          "description": "5v5 Ranked Flex ",
          "notes": null
      },
      {
          "queueId": 450,
          "map": "Howling Abyss",
          "description": "5v5 ARAM ",
          "notes": null
      },
      {
          "queueId": 600,
          "map": "Summoner's Rift",
          "description": "Blood Hunt Assassin ",
          "notes": null
      },
      {
          "queueId": 610,
          "map": "Cosmic Ruins",
          "description": "Dark Star: Singularity ",
          "notes": null
      },
      {
          "queueId": 700,
          "map": "Summoner's Rift",
          "description": "Clash ",
          "notes": null
      },
      {
          "queueId": 820,
          "map": "Twisted Treeline",
          "description": "Co-op vs. AI Beginner Bot ",
          "notes": null
      },
      {
          "queueId": 830,
          "map": "Summoner's Rift",
          "description": "Co-op vs. AI Intro Bot ",
          "notes": null
      },
      {
          "queueId": 840,
          "map": "Summoner's Rift",
          "description": "Co-op vs. AI Beginner Bot ",
          "notes": null
      },
      {
          "queueId": 850,
          "map": "Summoner's Rift",
          "description": "Co-op vs. AI Intermediate Bot ",
          "notes": null
      },
      {
          "queueId": 900,
          "map": "Summoner's Rift",
          "description": "URF ",
          "notes": null
      },
      {
          "queueId": 910,
          "map": "Crystal Scar",
          "description": "Ascension ",
          "notes": null
      },
      {
          "queueId": 920,
          "map": "Howling Abyss",
          "description": "Legend of the Poro King ",
          "notes": null
      },
      {
          "queueId": 940,
          "map": "Summoner's Rift",
          "description": "Nexus Siege ",
          "notes": null
      },
      {
          "queueId": 950,
          "map": "Summoner's Rift",
          "description": "Doom Bots Voting ",
          "notes": null
      },
      {
          "queueId": 960,
          "map": "Summoner's Rift",
          "description": "Doom Bots Standard ",
          "notes": null
      },
      {
          "queueId": 980,
          "map": "Valoran City Park",
          "description": "Star Guardian Invasion: Normal ",
          "notes": null
      },
      {
          "queueId": 990,
          "map": "Valoran City Park",
          "description": "Star Guardian Invasion: Onslaught ",
          "notes": null
      },
      {
          "queueId": 1000,
          "map": "Overcharge",
          "description": "PROJECT: Hunters ",
          "notes": null
      },
      {
          "queueId": 1010,
          "map": "Summoner's Rift",
          "description": "Snow ARURF ",
          "notes": null
      },
      {
          "queueId": 1020,
          "map": "Summoner's Rift",
          "description": "One for All ",
          "notes": null
      },
      {
          "queueId": 1030,
          "map": "Crash Site",
          "description": "Odyssey Extraction: Intro ",
          "notes": null
      },
      {
          "queueId": 1040,
          "map": "Crash Site",
          "description": "Odyssey Extraction: Cadet ",
          "notes": null
      },
      {
          "queueId": 1050,
          "map": "Crash Site",
          "description": "Odyssey Extraction: Crewmember ",
          "notes": null
      },
      {
          "queueId": 1060,
          "map": "Crash Site",
          "description": "Odyssey Extraction: Captain ",
          "notes": null
      },
      {
          "queueId": 1070,
          "map": "Crash Site",
          "description": "Odyssey Extraction: Onslaught ",
          "notes": null
      },
      {
          "queueId": 1090,
          "map": "Convergence",
          "description": "Teamfight Tactics ",
          "notes": null
      },
      {
          "queueId": 1100,
          "map": "Convergence",
          "description": "Ranked Teamfight Tactics ",
          "notes": null
      },
      {
          "queueId": 1110,
          "map": "Convergence",
          "description": "Teamfight Tactics Tutorial ",
          "notes": null
      },
      {
          "queueId": 1111,
          "map": "Convergence",
          "description": "Teamfight Tactics test ",
          "notes": null
      },
      {
          "queueId": 1300,
          "map": "Nexus Blitz",
          "description": "Nexus Blitz ",
          "notes": null
      },
      {
          "queueId": 1400,
          "map": "Summoner's Rift",
          "description": "Ultimate Spellbook ",
          "notes": null
      },
      {
          "queueId": 2000,
          "map": "Summoner's Rift",
          "description": "Tutorial 1",
          "notes": null
      },
      {
          "queueId": 2010,
          "map": "Summoner's Rift",
          "description": "Tutorial 2",
          "notes": null
      },
      {
          "queueId": 2020,
          "map": "Summoner's Rift",
          "description": "Tutorial 3",
          "notes": null
      }
    ],

    summonerSpellsIDs: {
      21: "SummonerBarrier.png",
      1: "SummonerBoost.png",
      14: "SummonerDot.png",
      3: "SummonerExhaust.png",
      4: "SummonerFlash.png",
      6: "SummonerHaste.png",
      7: "SummonerHeal.png",
      13: "SummonerMana.png",
      30: "SummonerPoroRecall.png",
      31: "SummonerPoroThrow.png",
      11: "SummonerSmite.png",
      39: "SummonerSnowURFSnowball_Mark.png",
      32: "SummonerSnowball.png",
      12: "SummonerTeleport.png",
      54: "Summoner_UltBook_Placeholder.png",
    },

    regionsValues: {
      "EUNE": "eun1",
      "EU West": "euw1",
      "Brazil": "br1",
      "Japan": "jp1",
      "Korea": "kr",
      "Latin America North": "la1",
      "Latin America South": "la2",
      "North America": "na",
      "Oceania": "oc1",
      "Russia": "ru",
      "Turkey": "tr1",

    },

    routingValues: {
      "europe": ["EUNE", "EU West", "Turkey", "Russia"],
      "asia": ["Korea", "Japan"],
      "AMERICAS": ["North America", "Brazil" , "Latin America North", "Latin America South", "Oceania"],
    },


    region: "EUNE",
    regionapi: "",
    continent: "",

    summoner: "",
    summonerID: "",
    summonerData: {
      profileIconId: 1,
    },
    summonerLeague: {},
    tier: "IRON",
    rank: "",
    wins: 0,
    losses: 0,
    lp: 0,
    winRating: 0,
    puuid: "",
    matchList: {},

    matchInformations: [],
    maps: [],
    descriptions: [],
    gameStarts: [],
    gameDurations: [],
    days: 0,
    summonerScore: {
    },
    myStyle: {
      backgroundColor: "",
    },
    matchPlayers: {},
    teams: {},

    bigDmg: {},
  }),

  methods: {
    async getSummoner (summoner) {
      this.matchInformations = [];
      this.maps = [],
      this.descriptions = [],
      this.summonerScore = {};
      this.riotkey = "RGAPI-c3ca8865-ec4b-4692-b063-808d5e371efa";
      
      var keys = Object.keys(this.routingValues);

      if (this.routingValues[keys[0]].includes(this.region)) {
        this.continent = keys[0];
      }

      this.regionapi = this.regionsValues[this.region];

      // Gets summoner data
      var summonerData = "https://" + this.regionapi + ".api.riotgames.com/lol/summoner/v4/summoners/by-name/" + summoner + "?api_key=" + this.riotkey;
      var counter = false;
      await this.axios.get(summonerData).then((response) => {
        this.summonerData = response.data;
        console.log(this.summonerData);
        this.puuid = this.summonerData.puuid;
      }).catch(error => {
        console.log(error.message);
        counter = true;
      })
      if (counter == false) {
        this.summonerID = this.summonerData.id;
        document.querySelector("#loading").style.display = "block";
        var matches = "https://" + this.continent + ".api.riotgames.com/lol/match/v5/matches/by-puuid/" + this.puuid + "/ids?start=0&count=10&api_key=" + this.riotkey;
        await this.getMatchIds(matches);
      }
    },

    async getMatchIds(matches) {
      await this.axios.get(matches).then((response) => {
        this.matchList = response.data;
        console.log(this.matchList);
      })
      await this.getMatchInformation();
    },

    async getMatchInformation() {
      for (let i = 0; i < this.matchList.length; i++) {
        // Gets informations about matches using match IDs
        var apiLink = "https://" + this.continent + ".api.riotgames.com/lol/match/v5/matches/" + this.matchList[i] + "?api_key=" + this.riotkey;
        await this.axios.get(apiLink).then((response) => {
          this.matchInformations[String(i)] = response.data;
        })
      }
      await this.getSummonerLeague();
    },

    async getSummonerLeague() {
      await this.axios.get("https://" + this.regionapi + ".api.riotgames.com/lol/league/v4/entries/by-summoner/" + this.summonerID + "?api_key=" + this.riotkey).then((response) => {
        this.summonerLeague = response.data;
        console.log("summonerLeague: ");
        console.log(this.summonerLeague);
      })
      this.logData();
    },

    logData: function () {


      for (let i = 0; i < this.matchInformations.length; i++) {
        for (let x = 0; x < this.matchInformations[i].info.participants.length; x++) {
          var puuid = this.matchInformations[i].info.participants[x].puuid;
          var scores =  this.matchInformations[i].info.participants[x];
          if (puuid== this.puuid) {
            this.summonerScore[i] = {
              kills: scores.kills,
              deaths: scores.deaths,
              assists: scores.assists,
              champName: scores.championName.toLowerCase(),
              champLevel: scores.champLevel,
              minions: scores.totalMinionsKilled + scores.neutralMinionsKilled,
              summoner1: scores.summoner1Id,
              summoner2: scores.summoner2Id,
            }

            if (scores.win) {
              this.summonerScore[i].win = scores.win;
              this.summonerScore[i].result = "Victory";
              this.myStyle[i] = {
                backgroundColor: "#91e0f2",
              }
            } else {
              this.summonerScore[i].win = scores.win;
              this.summonerScore[i].result = "Defeat";
              this.myStyle[i] = {
                backgroundColor: "#fa9da5",
              }
            }
          }
        }
        this.matchPlayers[i] = Object.assign(this.matchInformations[i].info.participants);
        this.teams[i] = Object.assign(this.matchInformations[i].info.teams);
        var dmg = 1;
        for (let j = 0; j < this.matchPlayers[i].length; j++) {
          if (this.matchPlayers[i][j].totalDamageDealtToChampions > dmg) {
            dmg = this.matchPlayers[i][j].totalDamageDealtToChampions;
          }
          
        }

        this.bigDmg[i] = {
          totalDmg: dmg,
        }

        if (this.teams[i][0].win) {
          this.teams[i][0].win = "Victory";
          this.teams[i][1].win = "Defeat";

        } else {
          this.teams[i][1].win = "Victory";
          this.teams[i][0].win = "Defeat";
        }
      }  

      
      this.gameStarts = [];
      for (let i = 0; i < this.matchInformations.length; i++) {
        for (let x = 0; x < this.queIDs.length; x++) {
          if (this.matchInformations[i].info.queueId == this.queIDs[x].queueId) {
            this.maps.push(this.queIDs[x].map);
            this.descriptions.push(this.queIDs[x].description);
          }
        }
        this.gameStarts.push(Math.round(((Date.now() - this.matchInformations[i].info.gameStartTimestamp) / 1000) / 3600));
        this.gameDurations.push(Math.round(this.matchInformations[i].info.gameDuration / 1000 / 60));
      }
      for (let i = 0; i < this.gameStarts.length; i++) {
        if (this.gameStarts[i] / 24 > 0.99) {
          this.gameStarts[i] = this.gameStarts[i] / 24;
          this.gameStarts[i] = Math.round(this.gameStarts[i]);
          this.gameStarts[i] = this.gameStarts[i].toString() + " Days ago";
        } else {
          this.gameStarts[i] = this.gameStarts[i].toString() + " Hours ago";
        }
      }
      console.log("summoner score");
      console.log(this.summonerScore);
      if (this.summonerLeague[0].queueType == "RANKED_SOLO_5x5") {
        this.wins = this.summonerLeague[0].wins;
        this.losses = this.summonerLeague[0].losses;
        this.tier = this.summonerLeague[0].tier;
        this.rank = this.summonerLeague[0].rank;
        this.lp = this.summonerLeague[0].leaguePoints;
      } else {
        this.wins = this.summonerLeague[1].wins;
        this.losses = this.summonerLeague[1].losses;
        this.tier = this.summonerLeague[1].tier;
        this.rank = this.summonerLeague[1].rank;
        this.lp = this.summonerLeague[1].leaguePoints;
      }

      this.winRating = (this.wins / (this.wins + this.losses)) * 100;
      this.winRating = Math.round(this.winRating);
      console.log(this.winRating);

      document.querySelector("#loading").style.display = "none";
      document.querySelector("#main").style.display = "block";
      console.log(this.matchInformations);
      console.log("Maps: ");
      console.log(this.maps);
      console.log("Descriptions: ");
      console.log(this.descriptions);

    }
  },
};
</script>
