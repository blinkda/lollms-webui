<template>
  <div class="container bg-bg-light dark:bg-bg-dark shadow-lg overflow-y-auto scrollbar-thin scrollbar-track-bg-light-tone scrollbar-thumb-bg-light-tone-panel hover:scrollbar-thumb-primary dark:scrollbar-track-bg-dark-tone dark:scrollbar-thumb-bg-dark-tone-panel dark:hover:scrollbar-thumb-primary active:scrollbar-thumb-secondary">
    <div class="container flex flex-row m-2">
      <div class="flex-grow m-2">
        <div class="flex-grow m-2 p-2 border border-blue-300 rounded-md border-2 border-blue-300 m-2 p-4">
          <label class="mt-2">Presets</label>
          <select v-model="selectedPreset" class="m-2 border-2 rounded-md shadow-sm w-full">
            <option v-for="preset in Object.keys(presets)" :key="preset" :value="preset">
              {{ preset }}
            </option>
          </select>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="setPreset"  title="Use preset"><i data-feather="check"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="addPreset"  title="Add this text as a preset"><i data-feather="plus"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="removePreset"  title="Remove preset"><i data-feather="x"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="savePreset"  title="Save presets list"><i data-feather="save"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="reloadPresets"  title="Reload presets list"><i data-feather="refresh-ccw"></i></button>
          
        </div>
        <div class="flex-grow m-2 p-2 border border-blue-300 rounded-md border-2 border-blue-300 m-2 p-4">
          <div class="m-0">
            <button v-show="!generating" id="generate-button" @click="generate" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="pen-tool"></i></button>
            <button v-show="generating" id="stop-button" @click="stopGeneration" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="x"></i></button>
            <button
                type="button"
                @click="startSpeechRecognition"
                :class="{ 'text-red-500': isLesteningToVoice }"
                class="w-6 hover:text-secondary duration-75 active:scale-90 cursor-pointer"
            >   
            <i data-feather="mic"></i>
            </button>
            <button
                    title="speak"
                    @click.stop="speak()"
                    :class="{ 'text-red-500': isTalking }"
                    class="w-6 hover:text-secondary duration-75 active:scale-90 cursor-pointer">
              <i data-feather="volume-2"></i>
            </button>
            <button v-show="!generating" id="export-button" @click="exportText" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="upload"></i></button>
            <button v-show="!generating" id="import-button" @click="importText" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="download"></i></button>
          
            <input type="file" id="import-input" class="hidden">
          </div>          
          <textarea v-model="text" id="text_element" class="mt-4 h-64 overflow-y-scroll w-full dark:bg-bg-dark  scrollbar-thin scrollbar-track-bg-light-tone scrollbar-thumb-bg-light-tone-panel hover:scrollbar-thumb-primary dark:scrollbar-track-bg-dark-tone dark:scrollbar-thumb-bg-dark-tone-panel dark:hover:scrollbar-thumb-primary active:scrollbar-thumb-secondary" type="text"></textarea>
          <span>Cursor position {{ cursorPosition }}</span>
                    
        </div>
        <div class="flex-grow m-2 p-2 border border-blue-300 rounded-md border-2 border-blue-300 m-2 p-4">
          <MarkdownRenderer ref="mdRender" :markdown-text="text" class="dark:bg-bg-dark">
          </MarkdownRenderer>          
        </div>
      </div>
      <div id="settings" class="border border-blue-300 bg-blue-200 mt-4 w-25 mr-2 h-full mb-10 min-w-500" style="align-items: center; height: fit-content; margin: 10px; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); border-radius: 4px;">
        <div id="title" class="border border-blue-600 bg-blue-300 m-0 flex justify-center items-center box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1) border-radius: 4px;">
          <h3 class="text-gray-600 mb-4 text-center m-0">Settings</h3>
        </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Temperature</h3>
            <input type="range" v-model="temperature" min="0" max="5" step="0.1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ temperature }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Top K</h3>
            <input type="range" v-model="top_k" min="1" max="100" step="1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ top_k }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Top P</h3>
            <input type="range" v-model="top_p" min="0" max="1" step="0.1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ top_p }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Repeat Penalty</h3>
            <input type="range" v-model="repeat_penalty" min="0" max="5" step="0.1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ repeat_penalty }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Repeat Last N</h3>
            <input type="range" v-model="repeat_last_n" min="0" max="100" step="1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ repeat_last_n }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Number of tokens to crop the text to</h3>
            <input type="number" v-model="n_crop" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ n_crop }}</span>
          </div>          
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Number of tokens to generate</h3>
            <input type="number" v-model="n_predicts" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ n_predicts }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Seed</h3>
            <input type="number" v-model="seed" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ seed }}</span>
          </div>
      </div>
    </div>
  </div>
  <Toast ref="toast"/>
</template>

<script>

import feather from 'feather-icons'
import axios from "axios";
import socket from '@/services/websocket.js'
import Toast from '../components/Toast.vue'
import MarkdownRenderer from '../components/MarkdownRenderer.vue';
export default {
  name: 'PlayGroundView',
  data() {
    return {
      generating:false,
      isSpeaking:false,
      voices: [],    
      isLesteningToVoice:false,
      presets:{},
      selectedPreset: '',    
      cursorPosition:0,  
      text:"",
      pre_text:"",
      post_text:"",
      temperature: 0.1,
      top_k: 50,
      top_p: 0.9,
      repeat_penalty: 1.3,
      repeat_last_n: 50,
      n_crop: -1,
      n_predicts: 2000,
      seed: -1,
    };
  },
  components:{    
    Toast,
    MarkdownRenderer
  },
  mounted() {
    const text_element = document.getElementById('text_element');
    text_element.addEventListener('input', () => {
      this.cursorPosition = text_element.selectionStart;
    });    
    text_element.addEventListener('click', () => {
      this.cursorPosition = text_element.selectionStart;
    });        
    axios.get('./presets.json').then(response => {
          console.log(response.data)
          this.presets=response.data
        }).catch(ex=>{
          this.$refs.toast.showToast(`Error: ${ex}`,4,false)
        });
        // Event handler for receiving generated text chunks
        socket.on('text_chunk', data => {
            this.appendToOutput(data.chunk);
            const text_element = document.getElementById('text_element');
            text_element.scrollTo(0, text_element.scrollHeight);
        });

        // Event handler for receiving generated text chunks
        socket.on('text_generated', data => {
            // Toggle button visibility
            this.generating=false;
        });

        socket.on('generation_error', data => {
            console.log('generation_error:', data);
            this.$refs.toast.showToast(`Error: ${data}`,4,false)
            // Toggle button visibility
            this.generating=false;
        });

        

        // Event handler for successful connection
        socket.on('connect', () => {
            console.log('Connected to LoLLMs server');
            this.$store.state.isConnected=true;
            this.generating=false
        });

        // Event handler for error during text generation
        socket.on('buzzy', error => {
            console.error('Server is busy. Wait for your turn', error);
            this.$refs.toast.showToast(`Error: ${error.message}`,4,false)
            // Toggle button visibility
            this.generating=false
        });

        // Event handler for error during text generation
        socket.on('generation_canceled', error => {
            // Toggle button visibility
            this.generating=false
            console.log("Generation canceled OK")
        });

      //console.log('chatbox mnt',this.$refs)
      this.$nextTick(() => {
          feather.replace();
      });  
      
    // Speach synthesis
    // Check if speech synthesis is supported by the browser
    if ('speechSynthesis' in window) {
    this.speechSynthesis = window.speechSynthesis;

    // Load the available voices
    this.voices = this.speechSynthesis.getVoices();

    // Make sure the voices are loaded before starting speech synthesis
    if (this.voices.length === 0) {
        this.speechSynthesis.addEventListener('voiceschanged', this.onVoicesChanged);
    } else {
    }
    } else {
    console.error('Speech synthesis is not supported in this browser.');
    }


  },
  created(){

        
  },
  computed: {
    isTalking :{
        get(){
            return this.isSpeaking
        }
    },
  },
  methods:{
    onVoicesChanged() {
      // This event will be triggered when the voices are loaded
      this.voices = this.speechSynthesis.getVoices();
      },
      speak() {
          if (this.msg) {
              this.speechSynthesis.cancel();
              this.msg = null;
              this.isSpeaking = false;
              return;
          }
          let startIndex =0;
          // Set isSpeaking to true before starting synthesis
          console.log("voice on")
          this.isSpeaking = true;

          const chunkSize = 200; // You can adjust the chunk size as needed

          // Create a new SpeechSynthesisUtterance instance
          this.msg = new SpeechSynthesisUtterance();
          this.msg.pitch = this.$store.state.config.audio_pitch;

          // Optionally, set the voice and other parameters as before
          if (this.voices.length > 0) {
              this.msg.voice = this.voices.filter(voice => voice.name === this.$store.state.config.audio_out_voice)[0];
          }


          // Function to find the index of the last sentence that fits within the chunk size
          const findLastSentenceIndex = (startIndex) => {
              let txt = this.text.substring(startIndex, startIndex+chunkSize)
              // Define an array of characters that represent end of sentence markers.
              const endOfSentenceMarkers = ['.', '!', '?'];

              // Initialize a variable to store the index of the last end of sentence marker.
              let lastIndex = -1;

              // Iterate through the end of sentence markers and find the last occurrence in the txt string.
              endOfSentenceMarkers.forEach(marker => {
              const markerIndex = txt.lastIndexOf(marker);
              if (markerIndex > lastIndex) {
                  lastIndex = markerIndex;
              }
              });
              if(lastIndex==-1){lastIndex=txt.length}
              console.log(lastIndex)
              return lastIndex+startIndex+1;
          };

          // Function to speak a chunk of text
          const speakChunk = () => {
              const endIndex = findLastSentenceIndex(startIndex);
              const chunk = this.text.substring(startIndex, endIndex);
              this.msg.text = chunk;
              startIndex = endIndex + 1;
              this.msg.onend = (event) => {
                  if (startIndex < this.text.length-2) {
                      // Use setTimeout to add a brief delay before speaking the next chunk
                      setTimeout(() => {
                          speakChunk();
                      }, 1); // Adjust the delay as needed
                  } else {
                      this.isSpeaking = false;
                      console.log("voice off :",this.text.length,"  ",endIndex)
                  }
              };
              this.speechSynthesis.speak(this.msg);
          };

          // Speak the first chunk
          speakChunk();
      },    
    getCursorPosition() {
      return this.cursorPosition;
    },    
    appendToOutput(chunk){
      this.pre_text += chunk
      this.text = this.pre_text + this.post_text
    },
    generate(){
      console.log("Finding cursor position")
      this.pre_text = this.text.substring(0,this.getCursorPosition())
      this.post_text = this.text.substring(this.getCursorPosition(), this.text.length)
      var prompt = this.text.substring(0,this.getCursorPosition())
      console.log(prompt)
      // Trigger the 'generate_text' event with the prompt
      socket.emit('generate_text', { prompt: prompt, personality: -1, n_predicts: this.n_predicts , n_crop: this.n_crop,
      parameters: {
          temperature: this.temperature,
          top_k: this.top_k,
          top_p: this.top_p,
          repeat_penalty: this.repeat_penalty, // Update with desired repeat penalty value
          repeat_last_n: this.repeat_last_n, // Update with desired repeat_last_n value
          seed: parseInt(this.seed)
      }});

      // Toggle button visibility
      this.generating=true
    },
    stopGeneration(){
      // Trigger the 'cancel_generation' event
      socket.emit('cancel_text_generation',{});
    },
    exportText(){
      const textToExport = this.text;
      const element = document.createElement('a');
      const file = new Blob([textToExport], {type: 'text/plain'});
      element.href = URL.createObjectURL(file);
      element.download = 'exported_text.txt';
      document.body.appendChild(element);
      element.click();
      document.body.removeChild(element);      
    },
    importText() {
      const inputFile = document.getElementById("import-input");
      if (!inputFile) return; // If the element doesn't exist, do nothing
      inputFile.addEventListener("change", event => {
        if (event.target.files && event.target.files[0]) {
          const reader = new FileReader();
          reader.onload = () => {
            this.text = reader.result;
          };
          reader.readAsText(event.target.files[0]);
        } else {
          alert("Please select a file.");
        }
      });
      inputFile.click();
    },
    setPreset() {
      this.text = this.presets[this.selectedPreset];
    },
    addPreset() {
      let title = prompt('Enter the title of the preset:');
      this.presets[title] =  this.text
    },
    removePreset() {
      if (this.selectedPreset) {
        delete this.presets[this.selectedPreset];
      }
    },
    savePreset() {
      axios.post("/save_presets", this.presets).then((response) => {
          console.log(response);
          this.$refs.toast.showToast(`Presets saved`,4,true)
        });
    },
    reloadPresets() {
      axios.get('./presets.json').then(response => {
          console.log(response.data)
          this.presets=response.data
        }).catch(ex=>{
          this.$refs.toast.showToast(`Error: ${ex}`,4,false)
        });
    },
    startSpeechRecognition() {
        if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
            this.recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
            this.recognition.lang = this.$store.state.config.audio_in_language; // Set the language, adjust as needed
            this.recognition.interimResults = true; // Enable interim results to get real-time updates

            this.recognition.onstart = () => {
              this.isLesteningToVoice = true;
              this.silenceTimer = setTimeout(() => {
                  this.recognition.stop();
              }, this.silenceTimeout); // Set the silence timeout to stop recognition
            };

            this.pre_text = this.text.substring(0,this.getCursorPosition())
            this.post_text = this.text.substring(this.getCursorPosition(), this.text.length)

            this.recognition.onresult = (event) => {
              this.generated = '';

              for (let i = event.resultIndex; i < event.results.length; i++) {
                this.generated += event.results[i][0].transcript;
              }
              this.text = this.pre_text + this.generated + this.post_text; // Update the textarea with the real-time recognized words
              this.cursorPosition = this.pre_text.length + this.generated.length;
              clearTimeout(this.silenceTimer); // Clear the silence timeout on every recognized result
              this.silenceTimer = setTimeout(() => {
                  this.recognition.stop();
              }, this.silenceTimeout); // Set a new silence timeout after every recognized result
            };

            this.recognition.onerror = (event) => {
            console.error('Speech recognition error:', event.error);
            this.isLesteningToVoice = false;
            clearTimeout(this.silenceTimer); // Clear the silence timeout on error
            };

            this.recognition.onend = () => {
              console.log('Speech recognition ended.');
              this.isLesteningToVoice = false;
              this.pre_text = this.pre_text + this.generated;
              this.cursorPosition = this.pre_text.length;
              clearTimeout(this.silenceTimer); // Clear the silence timeout when recognition ends normally
            };

            this.recognition.start();
        } else {
            console.error('Speech recognition is not supported in this browser.');
        }
        },
  }
};
</script>

<style>

  select {
    width: 200px;
  }

  body {
    background-color: #fafafa;
    font-family: sans-serif;
  }

  .container {
    margin: 4px auto;
    width: 800px;
  }

  .settings {
    position: fixed;
    top: 0;
    right: 0;
    width: 250px;
    background-color: #fff;
    z-index: 1000;
    display: none;
  }

  .settings-button {
    cursor: pointer;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    color: #333;
    font-size: 14px;
  }

  .settings-button:hover {
    background-color: #eee;
  }

  .settings-button:active {
    background-color: #ddd;
  }

  .slider-container {
    margin-top: 20px;
  }

  .slider-value {
    display: inline-block;
    margin-left: 10px;
    color: #6b7280;
    font-size: 14px;
  }

  .small-button {
    padding: 0.5rem 0.75rem;
    font-size: 0.875rem;
  }
</style>
  