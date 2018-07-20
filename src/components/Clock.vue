<template>
    <div class="main-card">
        <md-card>
            <md-card-header class="header">
                <md-card-header-text>
                    <div class="md-title">
                        <md-icon class="alarm">alarm</md-icon>
                        <span>Pomodoro Clock</span>
                    </div>
                    <div class="md-subhead">Write a blog article ...</div>
                </md-card-header-text>
            </md-card-header>
            <md-card-content class="controls">
                <div class="break">
                    <p>break length</p>
                    <md-button class="md-icon-button" @click.native="breakLengthChange(-1)">
                        <md-icon>remove</md-icon>
                    </md-button>
                    <span class="time">{{breakInterval}}</span>
                    <md-button class="md-icon-button" @click.native="breakLengthChange(1)">
                        <md-icon>add</md-icon>
                    </md-button>
                </div>
                <div class="session">
                    <p>session length</p>
                    <md-button class="md-icon-button" @click.native="sessionLengthChange(-1)">
                        <md-icon>remove</md-icon>
                    </md-button>
                    <span class="time">{{sessionInterval}}</span>
                    <md-button class="md-icon-button" @click.native="sessionLengthChange(1)">
                        <md-icon>add</md-icon>
                    </md-button>
                </div>
            </md-card-content>
            <md-card-content class="clock">
                <md-button class="md-fab" @click.native="startProgress">
                    <div class="time_type">{{session ? 'Session' : 'Break'}}</div>
                    <div class="time_left">{{timeToLeft}}</div>
                </md-button>
                <md-progress-spinner :md-diameter="175" :md-stroke="20" :md-value="progress"/>
            </md-card-content>
            <md-card-area md-inset>
                <md-card-content class="round">
                    <md-icon class="list" title="Tasks list">list</md-icon>
                    Today <md-chip>{{round}}/10</md-chip>
                    <md-icon class="settings" title="Settings">settings</md-icon>
                </md-card-content>
            </md-card-area>
            <md-card-actions>
                <md-button @click.native="pauseProgress">
                    <md-icon>{{pause_icon}}</md-icon>
                    <span>{{pause_label}}</span>
                </md-button>
                <md-button @click.native="stopProgress">
                    <md-icon>stop</md-icon>
                    <span>Stop</span>
                </md-button>
            </md-card-actions>        
        </md-card>
        <div class="footer">
            Powered by <a href="https://vuejs.org/" target="blank">Vue.js</a> 
            and <a href="https://vuematerial.io/getting-started" target="blank">Vue Material</a>.
        </div>    
    </div>
</template>

<script>
export default {
  name: 'clock',
  data: () => ({
    progress: 0,
    progressInterval: null,
    progressPaused: false,
    timeToLeft: '25',
    sessionInterval: 25,
    breakInterval: 5,
    pause_label: 'Pause',
    pause_icon: 'pause',
    timeToEnd: 0,
    session: true,
    round: 0,
  }),
  methods: {
    updateTimeToLeft(time) {
        const d = new Number(time);
        const h = Math.floor(d / 3600);
        const m = Math.floor(d % 3600 / 60);
        const s = Math.floor(d % 3600 % 60);
        return ((h > 0 ? h + ":" + (m < 10 ? "0" : "") : "") + m + ":" + (s < 10 ? "0" : "") + s);
    },  
    startProgress() {
        // return when timer is in progress
        if (this.progressInterval) return;
        // initial value depends on sesion interval
        this.timeToEnd = this.sessionInterval * 60;

        this.progressInterval = window.setInterval(() => {
            if (this.progressPaused) return;

            if (this.session) {
                // calculate progress portion
                this.progress += 100 / (this.sessionInterval * 60);
                // update time to left
                this.timeToLeft = this.updateTimeToLeft(--this.timeToEnd);
                // switch to break countdown
                if (this.progress >= 100) {
                    this.session = false;
                    this.timeToEnd = this.breakInterval * 60;
                    this.timeToLeft = this.breakInterval;
                    this.round++;
                }
            } else if (!this.session) {
                // calculate progress portion
                this.progress -= 100 / (this.breakInterval * 60);
                // update time to left
                this.timeToLeft = this.updateTimeToLeft(--this.timeToEnd);
                if (this.progress <= 0) {
                    // restart pomodoro timer
                    this.restartSession();
                }
            }
      }, 1000); // repeat every second
    },
    restartSession() {
        this.session = true;
        this.progressPaused = false;
        this.progress = 0;
        this.timeToEnd = (this.sessionInterval * 60);
        this.timeToLeft = this.sessionInterval;
    },
    restartBreak() {
        this.session = false;
        this.progressPaused = false;
        this.progress = 100;
        this.timeToEnd = (this.breakInterval * 60);
        this.timeToLeft = this.breakInterval;
    },
    pauseProgress() {
        this.progressPaused = !this.progressPaused;  
        if (this.progressPaused) {
            this.pause_label = 'Continue';
            this.pause_icon = 'play_arrow';
        } else {
            this.pause_label = 'Pause';
            this.pause_icon = 'pause';
        }
    },
    stopProgress() {
        window.clearInterval(this.progressInterval);
        this.progressInterval = null;
        this.restartSession();
    },
    sessionLengthChange(val) { 
        this.sessionInterval += val;
        if (this.sessionInterval < 1) {
            this.sessionInterval = 1;
        }
        if (this.session) {
            this.restartSession();
            this.timeToEnd = (this.sessionInterval * 60);
            this.timeToLeft = this.sessionInterval;
        }
    },
    breakLengthChange(val) { 
        this.breakInterval += val;
        if (this.breakInterval < 1) {
            this.breakInterval = 1;
        }
        if (!this.session && this.progressInterval) {
            this.restartBreak();
            this.timeToEnd = (this.breakInterval * 60);
            this.timeToLeft = this.breakInterval;
        }        
    },
  }
}
</script>

<style>
.main-card {
    width: 320px;
    margin-top: 60px;
}
.alarm {
    padding-bottom: 5px;
}
.header {
    background-color: #64dd17;
    color: #424242;
    text-align: left;
}
.clock {
    position: relative;
    width: 100%;
    height: 200px;
    text-align: center;
}
.clock .md-fab {
    margin: 9px;
    border-radius: 50%;
    width: 150px;
    height: 150px;
    padding-bottom: 0;
    color: #424242 !important;
}
.clock .time_type {
    font-size: 17px;
    line-height: 20px;
}
.clock .time_left {
    font-size: 30px;
}
.clock .md-progress-spinner {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
.clock .md-progress-spinner-circle {
    stroke: whitesmoke !important;
}
.controls {
    display: flex;
    padding: 0;
}
.break, .session {
    padding-left: 10px;
}
.break p, .session p {
    font-size: 15px;
    text-transform: uppercase;
    text-align: center;
}
.time {
    font-size: 1.5em;
    padding-left: 10px;
    padding-right: 10px;
    display: inline-block;
    padding-top: 10px;
}
.round {
    text-align: center;
    padding-top: 0;
}
.round .md-chip {
    font-size: 15px;
}
.md-icon.list  {
    position: absolute;
    left: 15px;
    top: 5px;
    cursor: pointer;
}
.md-icon.settings {
    position: absolute;
    right: 15px;
    top: 5px;
    cursor: pointer;
}
.footer {
    margin-top: 10px;
    text-align: center;
    font-style: italic;
    color: #64dd17;
}
</style>
