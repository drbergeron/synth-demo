<template>
  <v-container>
    <v-row align="center">
      <v-col cols="auto" >
        <v-select full-width:True ref="voicelist" v-model="selectedvoice" :items="voices"  label="Voice" item-text="name" item-value="voiceURI"  return-object ></v-select>
      </v-col>
    </v-row>
    <v-row align="center" justify="center">
      <v-col>
        <v-slider
          v-model="volumeValue"
          persistent-hint
          max="1"
          step=".1"
          hint="volume"
          thumb-label
          dense
        ></v-slider>
      </v-col>
      <v-col>
        <v-slider
          v-model="rateValue"
          persistent-hint
          max="2"
          min="1"
          step=".1"
          thumb-label
          hint="speech rate"
          dense
          
        ></v-slider>
      </v-col>
      <v-col>
        <v-slider
          v-model="pitchValue"
          persistent-hint
          max="2"
          step=".2"
          thumb-label
          hint="pitch"
          dense
          
        ></v-slider>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <button
        @click="initClient()"
        color="green"
        outlined
        >Start Process</button>
      </v-col>
      <v-col>
        <button @click="CancelCurrent()" outlined > Cancel </button>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        backlog count {{ backlog.length }}
      </v-col>
      <v-col>
        <v-card v-for="comment in displayed" :key="comment.id" >
          <v-row> 
            <v-col>
              {{ comment.speech }}
            </v-col>
          </v-row>
        </v-card>
      </v-col>
      <v-col v-if="isVoiceInitialized">
        isSpeeking gate: {{ speechInProgress }} <br/>
        isProcessing gate: {{isProcessingCurrentComment}}<br/>
        <b>Synth Status:</b> <br/>
        paused:    {{ synth.paused }} <br/>
        pending:   {{ synth.pending }} <br/>
        speaking:   {{ synth.speaking }}<br/>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'synthDemo',

  data: () => ({
    debuglog: true,
    displayed: [],
    voices: [],
    backlog: [],
    speechInProgress: false,
    synth: null,
    selectedvoice: null,
    volumeValue: .8,
    rateValue: 1.2,
    pitchValue: 1,
    currentSpeechStartTime: null,
    isProcessingCurrentComment: false,
    isVoiceInitialized: false,
    shortSpeech: " this is a short speech ",
    mediumSpeech: " this is a medium length speech and probably contains a bunch of other stuff that's relevant and will make it last between 6 and 9 seconds, probably not up to 42 seconds but you'll never know",
    longSpeech: " Move some NWIPA to the brite, brew Amber and more IPA which, shit I forgot to order malt for, ah no worries it's at the end of the week. Cleaning and general repairs. Probably time to put the new proper fuse back into the second cold room chiller instead of that automotive fuse we kludged in there... See about getting some new LVP arranged for the mezzanine floor. Eight years of \"Enh that fir decking will be fine until we figure something better out\" has resulted in some very well loved floorboards. Since it's been closed to the public for months might as well spruce it up for their eventual return someday.",
    speechCounter: 0,
    isStarted: false,
    clientDeviceSpeechTimeout: 9000
  }),

  mounted(){
    this.isVoiceInitialized = false;
    this.synth = this.win.speechSynthesis;
    this.isAndroid = this.IsAndroid();
    this.isMobileOrFirefox = this.IsMobileOrFF();
  },

  methods: {
    initClient(){
      this.PopulateVoices();
      this.SimulateComments();
    },

    IsMobileOrFF(){
      let check = false;
      (function(a){if(/(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino/i.test(a)||/1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(a.substr(0,4))) check = true;})(navigator.userAgent||navigator.vendor||window.opera); // eslint-disable-line 
      if (!check)
        check = navigator.userAgent.toLowerCase().indexOf('firefox') > -1;
      return check;
    },

    AddCommentToBacklog(message){
      
  
      this.backlog.push(message);
      
    },

    SimulateComments(){
      if(!this.isStarted){
        this.isStarted = true;
        this.DebugLog("starting comment generation process");
        var process = setInterval(() => {  // eslint-disable-line 
          this.DebugLog("generatingComment " + this.speechCounter);
          this.speechCounter++;
          let newComment = null;
          if( this.speechCounter % 3 === 1){
            this.DebugLog("mod 0");
            newComment = {
              id: this.speechCounter,
              speech: this.speechCounter + this.shortSpeech
            }
          }else if(this.speechCounter % 3 === 2){
             this.DebugLog("mod 1");
             newComment = {
              id: this.speechCounter,
              speech: this.speechCounter + this.mediumSpeech
            }
          }else if(this.speechCounter % 3 === 0){
             this.DebugLog("mod 2");
             newComment = {
              id: this.speechCounter,
              speech: this.speechCounter + this.longSpeech,
            }
          }

          if(this.backlog < 10){
            this.AddCommentToBacklog(newComment);
            this.ProcessComments();
   
          }

        }, 6000)
      }else{
        this.DebugLog("process already started");
      }
    },

    IsAndroid(){
      var ua = navigator.userAgent.toLowerCase();
      return ua.indexOf("android") > -1;
    },

    ProcessComments(){
      this.DebugLog("2 --------------------- ProcessComments");
      
      if(this.speechInProgress === true || this.isProcessingCurrentComment || this.backlog.length === 0){
        //if speech is in progress, check to see if it's long running?
        return false;
      }

      this.isProcessingCurrentComment = true;
      let currentcomment = this.backlog.shift();
      this.PushToFeed(currentcomment);
      this.DebugLog("current comment:");
      this.DebugLog(currentcomment);
      this.SayComment(currentcomment);
     
    },

    PushToFeed(updComment){
      
      this.DebugLog("6 - running PushToFeed ");

      this.displayed.unshift(updComment); //put comment into front
      if(this.displayed.length > 10)
        this.displayed = this.displayed.slice(0,10); //slam out anything over 10  
    },

    SayComment(comment){
      this.DebugLog("running say comment " + comment.speech);
      var toSpeak = new SpeechSynthesisUtterance(comment.speech);
      toSpeak.onend = function(event){   // eslint-disable-line 
        //kill the keep alive voice timer
        this.DebugLog("running speech callback to clear timeout");
        clearTimeout(this.speechTimer);
        this.speechInProgress = false;
        this.isProcessingCurrentComment = false;
        //now that we're done with this speech, kick off the process again
        this.ProcessComments();
      }.bind(this);

      
      toSpeak.voice = this.selectedvoice;
      toSpeak.lang = this.selectedvoice.lang;
      toSpeak.rate = this.rateValue; // From 0.1 to 10
      toSpeak.volume = this.volumeValue;
      toSpeak.pitch = this.pitchValue;

      this.currentSpeechStartTime = new Date().getTime();
      try{
        this.speechInProgress = true;
        this.synth.speak(toSpeak);
        //make a timer for this speech that times out after 9 seconds
        this.speechTimer = setTimeout(this.SpeechTimerAction, this.clientDeviceSpeechTimeout);
        //process the utterance
        
      }catch(e){
        //error speeking, cancel current JIC and then set progress to false 
        this.CancelCurrent();
      }
    },

     IsSpeechLongRunning(){
      this.DebugLog("5 - check long running speech");

      if(this.speechInProgress ){
        //if it's been longer than 30 seconds pause/resume to see if that fixes it
        var elapsed = new Date().getTime() - this.currentSpeechStartTime;
        if(elapsed / 1000 > this.commentMaxTimeSec){
          return true;
        }else{
          return false;
        }
      }else{
        //speech isn't running, return false
        return false;
      }
    },

    SpeechTimerAction(){
    
      if(this.IsSpeechLongRunning()){
        this.DebugLog("killing speech, long running");
        this.CancelCurrent();
      }else{
        if(!this.isMobileOrFirefox){
          this.DebugLog("quick-pausing ");
          this.win.speechSynthesis.pause();
          this.win.speechSynthesis.resume();
        }
        this.speechTimer = setTimeout(this.SpeechTimerAction, this.clientDeviceSpeechTimeout);
      }
    },

    PopulateVoices(){
      var timer = setInterval(() => {
        this.voices = this.synth.getVoices();

        if(this.voices.length !== 0){

          this.DebugLog("lclvoice" + localStorage.voicePref);
          if(localStorage.voicePref && this.voices.find(v => v.name === localStorage.voicePref)){
            this.DebugLog("using saved voice");
            this.selectedvoice = this.voices.find(v => v.name === localStorage.voicePref);
          }else{
            this.selectedvoice = this.voices.find(v => v.default === true);
            if(!this.selectedvoice)
              this.selectedvoice = this.voices[0];     
          }

          this.DebugLog("selected voice " + this.selectedvoice.name) ;
          this.isVoiceInitialized = true;
          clearInterval(timer);
          this.DebugLog(this.voices);        
        }
      }, 1000);
    },

    IsCommentInBacklog(id){
      return this.backlog.some( msg => msg.id === id);
    },

    IsCommentInDisplay(id){
      return this.displayed.some( msg => msg.id === id);
    },

    CancelCurrent(){
      this.DebugLog("****** Cancel current")
      clearTimeout(this.speechTimer);
      this.synth.cancel();
      this.speechInProgress = false;
      this.ProcessComments();
    },

    DebugLog(msg){
      if(this.debuglog)
        console.log(msg);
    },
  }
}
</script>
