<template>
    <div id="app">
        <header>
            <template v-for="p, i in currentGame.players">
                <input type="text" v-model="p.name" class="player-name"
                       :placeholder="'Player ' + (i + 1)">
            </template>
        </header>
        <div ref="body" class="body">
            <table ref="ends-table" class="ends">
                <tr v-for="end, endNumber in ends">
                    <template v-for="score, i in end">
                        <td>{{ score }}</td>
                        <td v-if="i == 0" class="end-number">{{ endNumber + 1}}</td>
                    </template>
                </tr>
            </table>
        </div>
        <footer>
            <div class="total-row">
                <div class="player-total-container" v-for="p, i in currentGame.players">
                    <span class="player-total">{{ playerTotal(p) }}</span>
                    <span class="end-change">(+{{ p.endScore }})</span>

                    <div class="score-buttons">
                        <button @click="updateScore(i, 1)" class="score-button">+</button>
                        <button @click="updateScore(i, -1)" class="score-button">-</button>
                    </div>
                </div>
            </div>
            <div class="confirm-row">
                <button @click="confirmScores" class="confirm-button">Confirm end</button>
                <button @click="removeEnd">Remove end</button>
                <button @click="restart">Restart game</button>
            </div>
        </footer>
    </div>
</template>

<script>

let STORAGE_KEY = 'bowls-0.1';
let storage = {
    fetch: () => {
        let savedGame = window.localStorage.getItem(STORAGE_KEY);
        return savedGame ? JSON.parse(savedGame) : makeGame();
    },
    save: (game) => {
        window.localStorage.setItem(STORAGE_KEY, JSON.stringify(game));
    }
}

let makePlayer = () => ({
    name: '',
    endScore: 0,
    ends: [],
});

let makeGame = () => ({
    players: [makePlayer(), makePlayer()]
});

export default {
    name: 'app',
    data: () => {
        return {
            currentGame: storage.fetch()
        };
    },
    computed: {
        ends() {
            // Get minimum length of all player's end scores list.
            let length = this.currentGame.players
                .map(p => p.ends.length)
                .reduce((a, b) => b < a ? b : a);
            let ends = [];
            for (let i = 0; i < length; i++) {
                let end = [];
                for (let player of this.currentGame.players) {
                    end.push(player.ends[i]);
                }
                ends.push(end);
            }
            return ends;
        }
    },
    methods: {
        playerTotal(player) {
            return player.ends.reduce((a, b) => a + b, 0);
        },
        updateScore(i, amount) {
            for (let j = 0; j < this.currentGame.players.length; j++) {
                if (j == i) continue;
                this.currentGame.players[j].endScore = 0;
            }
            this.currentGame.players[i].endScore = (
                Math.max(this.currentGame.players[i].endScore + amount, 0)
            );
        },
        removeEnd() {
            if (window.confirm('Are you sure you want to remove the last end?')) {
                for (let player of this.currentGame.players) {
                    player.endScore = player.ends[player.ends.length - 1];
                    player.ends = player.ends.slice(0, player.ends.length - 1);
                }
            }
        },
        confirmScores() {
            for (let player of this.currentGame.players) {
                player.ends.push(player.endScore);
                player.endScore = 0;
            }
        },
        restart() {
            if (window.confirm('Are you sure you want to restart?')) {
                this.currentGame = makeGame();
            }
        }
    },
    watch: {
        currentGame: {
            handler: function(game) {
                console.log("saving...");
                storage.save(game);
            },
            deep: true
        },
        ends() {
            // Scroll to bottom of ends listing when it changes.
            window.setTimeout(() => {
                let body = this.$refs.body;
                body.scrollTop = body.scrollHeight;
            });
        }
    }
}
</script>

<style>
    html, body {
        width: 100%;
        height: 100%;
        margin: 0;
    }

    .container {
        width : 100%;
        max-width: 640px;
    }

    header, .body, footer {
        position: absolute;
        max-width: 640px;
        width: 100%;
        left: 0;
        right: 0;
        margin: 0 auto 0 auto;
    }

    header {
        top: 0;
        height: 60px;
        overflow: hidden;
    }

    .body {
        top: 60px;
        bottom: 120px;
        overflow: auto;
    }

    footer {
        bottom: 0;
        overflow: hidden;
    }

    header > input {
        box-sizing: border-box;
        display: inline-block;
        margin: 0;
        width: 50%;
        height: 100%;
        font-size: 48px;
        text-align: center;
    }

    .ends {
        width: 100%;
    }

    .ends td {
        font-size: 24px;
        text-align: center;
    }

    .ends td.end-number {
        width: 3em;
        color: #888;
    }

    .player-total-container {
        box-sizing: border-box;
        display: inline-block;
        width: 50%;
        padding: 8px;
    }

    .player-total {
        font-size: 32px;
    }

    .end-change {
        font-size: 18px;
    }

    .score-buttons {
        display: inline-block;
        float: right;
    }

    .score-buttons > button {
        width: 50px;
        height: 50px;
    }
</style>
