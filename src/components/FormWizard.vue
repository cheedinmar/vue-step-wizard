<template>
    <div class="vue-step-wizard">
        <div class="step-header">
        <div class="step-progress">
            <div class="bar progressbar" :style="{ width: progress + '%' }">
            </div>
        </div>
        <ul class="step-pills">
            <li @click.prevent.stop="selectTab(index)" class="step-item" :class="{ 'active': tab.isActive, 'validated': tab.isValidated }" v-for="(tab, index) in tabs" v-bind:key="`tab-${index}`">
                <a class="step-link" href="#">
                        <span class="tabStatus">{{index+1}} </span> 
                        <span class="tabLabel">{{tab.title}}</span>
                          <svg xmlns="http://www.w3.org/2000/svg" class="check b" :class="{'validated' : tab.isValidated}"  viewBox="0 0 48 48" width="48px" height="48px"><linearGradient id="IMoH7gpu5un5Dx2vID39Ra" x1="9.858" x2="38.142" y1="9.858" y2="38.142" gradientUnits="userSpaceOnUse"><stop offset="0" stop-color="#9dffce"/><stop offset="1" stop-color="#50d18d"/></linearGradient><path fill="url(#IMoH7gpu5un5Dx2vID39Ra)" d="M44,24c0,11.045-8.955,20-20,20S4,35.045,4,24S12.955,4,24,4S44,12.955,44,24z"/><linearGradient id="IMoH7gpu5un5Dx2vID39Rb" x1="13" x2="36" y1="24.793" y2="24.793" gradientUnits="userSpaceOnUse"><stop offset=".824" stop-color="#135d36"/><stop offset=".931" stop-color="#125933"/><stop offset="1" stop-color="#11522f"/></linearGradient><path fill="url(#IMoH7gpu5un5Dx2vID39Rb)" d="M21.293,32.707l-8-8c-0.391-0.391-0.391-1.024,0-1.414l1.414-1.414	c0.391-0.391,1.024-0.391,1.414,0L22,27.758l10.879-10.879c0.391-0.391,1.024-0.391,1.414,0l1.414,1.414	c0.391,0.391,0.391,1.024,0,1.414l-13,13C22.317,33.098,21.683,33.098,21.293,32.707z"/></svg>
                      
                       
                </a>
            </li>
        </ul>
        </div>
        <div class="step-body">
            <form>
                <slot></slot>
            </form>
        </div>
        <div class="step-footer">
            <div class="btn-group" role="group">
                <template v-if="!submitSuccess">
                  <button @click="previousTab" :disabled="currentTab === 0" class="step-button step-button-previous">Previous</button>
                  <button @click="nextTab" v-if="currentTab < totalTabs - 1" class="step-button step-button-next">Next</button>
                  <button @click="onSubmit" v-if="currentTab === totalTabs - 1" class="step-button step-button-submit">Register Customer</button>
                </template>
                <template v-else>
                  <button @click="reset" class="step-button step-button-reset">Reset</button>
                </template>
            </div>
        </div>
    </div>
</template>
<script>
import { store } from "./store.js";
export default {
    name: 'form-wizard',
    data(){
        return{
            tabs: [],
            currentTab : 0,
            totalTabs : 0,
            storeState: store.state,
            submitSuccess : false,
            progress: 0,
            isValidationSupport: false,
            tabNumber:localStorage.tabNumber
        }
    },
    mounted(){
            this.tabs = this.$children;
            this.totalTabs = this.tabs.length;
            this.currentTab = this.tabs.findIndex((tab) => tab.isActive === true);

    //         if (localStorage.tabNumber ) {
    //           this.tabs[localStorage.tabNumber].isActive = true;
    //             this.currentTab = localStorage.tabNumber;
    //   // console.log(JSON.parse(localStorage.data))
    // }else{
    //   localStorage.setItem("tabNumber","0")
    // }
            // Select first tab if none is marked selected
           if(localStorage.tabNumber === -1  ||localStorage.tabNumber == 0 ){  
                this.tabs[0].isActive = true;
                this.currentTab = 0;
                console.log(this.tabNumber)
            }
             if(localStorage.tabNumber == 1 && localStorage.tabNumber > 0  ){
              this.tabs[1].isActive = true;
                this.currentTab = 1;
            }else if(localStorage.tabNumber == 2 && localStorage.tabNumber > 0){
              this.tabs[2].isActive = true;
                this.currentTab = 2;
            }else if(localStorage.tabNumber == 3 && localStorage.tabNumber > 0){
              this.tabs[3].isActive = true;
                this.currentTab = 3;
            }else if(localStorage.tabNumber == 4 && localStorage.tabNumber > 0){
              this.tabs[4].isActive = true;
                this.currentTab = 4;
            }else if(localStorage.tabNumber == 5 && localStorage.tabNumber > 0){
              this.tabs[5].isActive = true;
                this.currentTab = 5;
            }
            
            //Setup Initial Progress
            this.progress = ((this.currentTab + 1) / this.totalTabs * 100);

    },

    updated(){
        /*
          Using this lifehook - since store variable gets updated after component is mounted
          isValidationSupport checks if user is looking to perform validation on each step
        */
        this.isValidationSupport = (Object.keys(this.storeState.v).length !== 0 && this.storeState.v.constructor === Object) ? true : false;
    },

    methods:{
        previousTab(){
            this._switchTab(this.currentTab - 1);

            this.$emit('onPreviousStep'); 
        },

        nextTab(){

            if(this._validateCurrentTab() === false)
                return;

            this._switchTab(this.currentTab + 1);    

            this.$emit('onNextStep');          
              
        },

        reset(){

           this.tabs.forEach(tab => {
             tab.isActive = false;
             tab.isValidated = false;
           });

           this._switchTab(0);
           this.submitSuccess = false;
           this.storeState.v.$reset();

           this.$emit('onReset');
        },

        changeStatus(){
            this.submitSuccess = true;
        },

        selectTab(index){
            //Only switch to filled previous tabs
            if(index < this.currentTab){
              this._switchTab(index);
              return;
            }

            if(this._validateCurrentTab() === false){
                return;
            }

            if(this.tabs[index - 1].isValidated === false){
                return;
            }

            this._switchTab(index);
            
        },


        onSubmit(){
            if(this._validateCurrentTab() === false)
                return;
            this.$emit('onComplete');
            localStorage.tabNumber = 0
        },

        _switchTab(index){
            //Disable all tabs
            this.tabs.forEach(tab => {
              tab.isActive = false;
            });

            this.currentTab = index;
            this.tabs[index].isActive = true;

            this.progress = ((this.currentTab + 1) / this.totalTabs * 100);
        },

        _validateCurrentTab(){
            if(!this.isValidationSupport)  //Check if user wants to validate 
                return true;

            this.storeState.v.$touch();

            if (this.storeState.v.$invalid) {
                this.tabs[this.currentTab].isValidated = false;
                return false;
            }

            this.tabs[this.currentTab].isValidated = true;

            return true;
        }
    },
    watch:{
       currentTab(){
          store.setCurrentTab(this.currentTab);
       },
       currentTab: {
      handler(currentTab, oldData) {
        localStorage.tabNumber = currentTab
      },
      deep: true
    }

   
    
  },
    
}
</script>
<style>

 .progressbar {
  -webkit-transition: width 1s ease;
  transition: width 1s ease;
}
.vue-step-wizard {
  background-color: #f7f8fc;
  width: 100%;
  margin: auto;
  padding: 40px;
}
.step-progress {
  height: 1rem;
  background: #fff;
  border-radius: 1rem;
  margin: 1rem 0;
}
.step-progress .bar {
  content: "";
  height: 0.8rem;
  border-radius: 1.5rem;
  background-color: #074a74;
}
.step-pills {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  background-color: #fff;
  -webkit-box-pack: justify;
  -ms-flex-pack: justify;
  justify-content: space-between;
  padding: 1rem;
  border-radius: 1rem;
  -webkit-box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
}
.step-pills .step-item {
  background-color: #074a746c;
  border-radius: 10px;
  position: relative;
  list-style-type: none;
  padding: 1rem 1.5rem;
}
.step-pills .step-item a {
  text-decoration: none;
  color: #7b7b7b;
}
.step-pills .step-item.active {
  border: 1px solid #074a74a8;
  background-color: #074a74a8;
}
.step-pills .step-item.validated {
  border: 1px solid #074a74;
  background-color: #074a74;
}
.step-body {
 
  margin-left: auto;
  padding: 1.2rem;
}

.step-footer {
  padding: 1rem;
  border-radius: 1rem;
}
.step-footer {
  margin-left: auto;
  margin: 1rem 0;
  text-align: center;
}
.step-button {
  font-weight: 700;
  line-height: 1;
  text-transform: uppercase;
  position: relative;
  max-width: 30rem;
  text-align: center;
  border: 1px solid;
  border-radius: 12px;
  color: #22292f;
  color: rgba(34, 41, 47, var(--text-opacity));
  padding: 0.5rem 1.25rem;
  font-size: 0.875rem;
  margin: 0.5rem;
  color: #fff;
  outline: none !important;
  -webkit-box-shadow: none !important;
  box-shadow: none !important;
}
.step-button-next {
  background-color: #074a74;
  padding: 1.3rem 2.25rem;
}
.step-button-previous {
  background-color: #074a74;
  padding: 1.3rem 2.25rem;
}
.step-button-previous:disabled {
  background-color: #919191;
  padding: 1.3rem 2.25rem;
}
.step-button-submit {
  background-color: #074a74;
  padding: 1.3rem 2.25rem;
}
.step-button-reset {
  background-color: #074a74;
}
.tabStatus {
  display: inline-block;
  width: 1.5rem;
  height: 1.5rem;
  margin-right: 0.5rem;
  line-height: 1.5rem;
  color: #fff;
  text-align: center;
  background: rgba(0, 0, 0, 0.38);
  border-radius: 50%;
}
.tabLabel {
  color: #fff;
}
.check{
  margin-left: 0.7rem;
   margin-right: 0.7rem;
   width: 1.5rem;
  height:1.5rem;
  display: none;
}
.check.validated{
  display: inline-block;
}


</style>