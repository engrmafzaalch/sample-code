<template>
  <b-modal
    :id="`dashcard-${focus.focusId}`"
    modal-class="dashcard"
    hide-footer
    hide-header
    centered
    size="xl"
  >
    <div class="row">
      <div class="col-xl-9 bg-grey p3040">
        <div class="header-title">
          <h2 class="title">
            <span>I shall</span> {{ focus.destination.text1 }}
            <span>so that</span>
            {{ focus.destination.text2 }}
          </h2>
          <span
            class="ico ico__edit"
            @click="show.sidePanel = 'EDIT_FOCUS'"
            v-b-tooltip.hover
            title="Edit Focus"
          >
            <svg
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                fill-rule="evenodd"
                clip-rule="evenodd"
                d="M4 5C3.73478 5 3.48043 5.10536 3.29289 5.29289C3.10536 5.48043 3 5.73478 3 6V20C3 20.2652 3.10536 20.5196 3.29289 20.7071C3.48043 20.8946 3.73478 21 4 21H18C18.2652 21 18.5196 20.8946 18.7071 20.7071C18.8946 20.5196 19 20.2652 19 20V14.66C19 14.1077 19.4477 13.66 20 13.66C20.5523 13.66 21 14.1077 21 14.66V20C21 20.7957 20.6839 21.5587 20.1213 22.1213C19.5587 22.6839 18.7957 23 18 23H4C3.20435 23 2.44129 22.6839 1.87868 22.1213C1.31607 21.5587 1 20.7957 1 20V6C1 5.20435 1.31607 4.44129 1.87868 3.87868C2.44129 3.31607 3.20435 3 4 3H9.34C9.89228 3 10.34 3.44772 10.34 4C10.34 4.55228 9.89228 5 9.34 5H4Z"
                fill="#557EFB"
              />
              <path
                fill-rule="evenodd"
                clip-rule="evenodd"
                d="M17.2929 1.29289C17.6834 0.902369 18.3166 0.902369 18.7071 1.29289L22.7071 5.29289C23.0976 5.68342 23.0976 6.31658 22.7071 6.70711L12.7071 16.7071C12.5196 16.8946 12.2652 17 12 17H8C7.44772 17 7 16.5523 7 16V12C7 11.7348 7.10536 11.4804 7.29289 11.2929L17.2929 1.29289ZM9 12.4142V15H11.5858L20.5858 6L18 3.41421L9 12.4142Z"
                fill="#557EFB"
              />
            </svg>
          </span>
        </div>
        <div class="btns-toogle">
          <ToogleButtons
            :focusId="focus.focusId"
            :spheresProp="focus.spheres"
          />
          <div class="spot-splash">
            <span :class="{ active: !splash_switch }">SPOTLIGHT</span>
            <label class="switch">
              <input v-model="splash_switch" type="checkbox" checked />
              <span class="slider round"></span>
            </label>
            <span :class="{ active: splash_switch }">SPLASH</span>
          </div>
          <button class="btn-round-white" @click="show.sidePanel = 'TOOLS'">
            + ADD TOOLS
          </button>
        </div>

        <div class="card-grid">

          <card
            v-for="aTool in focus.tools.filter(el => !el._deleted_at)"
            :key="aTool.tool_id"
            :title="aTool.text"
            :splash="splash_switch"
          >

            <div slot="spotlight">
              <base-web-component
                :tool="aTool"
                :focus="focus"
                :user="$store.getters['user/user']"
              />
            </div>
            <div slot="splash">Splash</div>
            <div slot="setup">
              <base-setup></base-setup>
            </div>
            <div slot="connect">
              <base-connect></base-connect>
            </div>
          </card>
          <card
            title="UTubeU"
            :splash="splash_switch"
          >

            <div slot="spotlight">
              <UTube :id="videoid.id" :type="videoid.type" :title="videoid.title" />
            </div>
            <div slot="splash">Splash</div>
            <div slot="setup" >
              <BaseSetupPlayer
                @idUpdated="idUpdated"
                :utube="videoid"
              />
            </div>
            <div slot="connect">
              <base-connect></base-connect>
            </div>
          </card>
        </div>
      </div>
      <div class="col-xl-3 p-0 bg-white">
        <div class="default" v-if="show.sidePanel == 'DEFAULT'">
          <default-side-bar
            :crews="focus.crew"
            :dates="dates"
            @toogleDropdown="toogledropDown"
            :status_dropdown="show.status_dropdown"
            :state="focus.state"
            @showAddCrew="show.sidePanel = 'ADD_CREW'"
            :editPopup="show.editPopup"
            @showEditPopup="toogleEditPopup"
            @editCrew="editCrew"
            @deleteCrew="deleteCrew"
            :engines="focus.engines"
            @show-add-engine="show.sidePanel = 'ADD_ENGINE'"
            @edit-engine="showEditEngine"
            @close-dashcard="$emit('close-dashcard')"
            @changeFocusState="changeFocusState"
          ></default-side-bar>
          <!-- :engines="engines" -->
        </div>
        <div class="add-crew" v-if="show.sidePanel == 'ADD_CREW'">

          <add-crew-sidebar
            @closeOverlay="show.overlay = false"
            @closeAddCrew="closeAddCrew"
            @toogleAddPopup="toogleAddPopup"
            :addPopup="showAddPopup"
            @updateCrews="updateCrews"
            :currentCrews="focus.crew"
            @close-dashcard="$emit('close-dashcard')"
          ></add-crew-sidebar>
        </div>
        <div class="add-crew" v-if="show.sidePanel == 'TOOLS'">
          <tools-sidebar @closeMe="show.sidePanel = 'DEFAULT'" @close-dashcard="$emit('close-dashcard')"></tools-sidebar>
        </div>
        <div class="add-crew" v-if="show.sidePanel == 'EDIT_FOCUS'">
          <edit-focus
            @closeMe="show.sidePanel == 'DEFAULT'"
            :focus="focus"
            @updateFocus="updateFocus"
            @close-dashcard="$emit('close-dashcard')"
          ></edit-focus>
        </div>
        <div class="add-crew" v-if="show.sidePanel == 'ADD_ENGINE'">
          <add-engine
            @closeMe="show.sidePanel = 'DEFAULT'"
            @show-add-new="showAddNewEngine"
            @show-edit-engine="showEditEngine"
            :engines="focus.engines"
            @close-dashcard="$emit('close-dashcard')"
          ></add-engine>
        </div>
        <div class="add-crew" v-if="show.sidePanel == 'EDIT_ENGINE'">
          <edit-engine
            @closeMe="show.sidePanel = 'ADD_ENGINE'"
            :engine="editEngine"
            :editMode="editMode"
            @add-engine="addEngine"
            @update-engine="updateEngine"
            @close-dashcard="$emit('close-dashcard')"
          ></edit-engine>
        </div>
        <div
          class="overlay"
          @click="hideAllPopups"
          v-if="show.status_dropdown"
        ></div>
      </div>
    </div>
    <div class="overlay" v-show="show.overlay" @click="overlayClicked"></div>
  </b-modal>
</template>

<script>
import VueApexCharts from "vue-apexcharts";
import BaseWebComponent from "@/components/tools/baseWeb";
import DefaultSideBar from "@/components/dashcard/components/DefaultSideBar";
import ToogleButtons from "@/components/dashcard/components/ToogleButtons";
import Card from "@/components/dashcard/components/Card";
import AddCrewSidebar from "@/components/dashcard/components/AddCrewSidebar";
import ToolsSidebar from "@/components/dashcard/components/ToolsSidebar";
import EditFocus from "@/components/dashcard/components/EditFocus";
import AddEngine from "@/components/dashcard/components/AddEngine";
import EditEngine from "@/components/dashcard/components/EditEngine";
import BaseSetup from "@/components/dashcard/components/BaseSetup";
import BaseConnect from "@/components/dashcard/components/BaseConnect";
import AddCrewPopup from "./components/AddCrewPopup";
import UTube from "./components/utube/UTube";
import BaseSetupPlayer from "./components/utube/BaseSetupPlayer";
export default {
  name: "Dashcard",
  props: {
    focus: {
      type: Object,
      default: () => {
        return {
          ishall: "excel",
          sothat: "fulfill my mission"
        };
      }
    }
  },
  data() {
    return {
      videoid: {
        id: '527877676',
        title: 'Racing bike',
        type: 'vimeo'
      },
      editEngine: {},
      editMode: false,
      dates: {
        start: new Date(2018, 0, 16), // Jan 16th, 2018
        end: new Date(2018, 0, 19) // Jan 19th, 2018
      },
      showAddPopup: false,
      show: {
        sidePanel: "DEFAULT",
        overlay: false,
        status_dropdown: false,
        addPopup: false,
        editPopup: false
      },
      days: ["S", "M", "T", "W", "T", "F", "S"],
      splash_switch: false,
      series: [67, 55, 44],
      chartOptions: {
        chart: {
          height: 350,
          type: "radialBar"
        },
        colors: ["#8F99FC", "#7BB0FF", "#52B6D6"],

        labels: ["Exercise", "Stand", "Move"]
      },
      status: "On Track",
      editIndex: -1
    };
  },
  methods: {
    idUpdated(date){
      this.videoid = date

    },
    showAddNewEngine() {
      this.editMode = false;
      this.editEngine = {};
      this.show.sidePanel = "EDIT_ENGINE";
    },
    addEngine(engine) {
      let engines = this.focus.engines;
      engines.push(engine);
      this.$store
        .dispatch("user/updateFocusEngines", {
          engines: engines,
          focus: this.focus
        })
        .then(() => {
          console.log("Added Engine!");
          this.show.sidePanel = "ADD_ENGINE";
        });
    },
    updateEngine(engine) {
      let engines = this.focus.engines;
      engines[this.editIndex].text = engine.text;
      engines[this.editIndex].time = engine.time;
      engines[this.editIndex].when = engine.when;
      console.log(engines);
      this.$store
        .dispatch("user/updateFocusEngines", {
          engines: engines,
          focus: this.focus
        })
        .then(() => {
          this.editEngine = {};
          this.editIndex = -1;
          this.editMode = false;
          console.log("Updated Engine!");
          this.show.sidePanel = "ADD_ENGINE";
        });
    },
    showEditEngine(engine, i) {
      this.editEngine = engine;
      this.editIndex = i;
      this.editMode = true;
      this.show.sidePanel = "EDIT_ENGINE";
    },
    updateFocus(updatedFocus) {
      this.show.sidePanel = "DEFAULT";
      this.focus.destination.text1 = updatedFocus.ishall;
      this.focus.destination.text2 = updatedFocus.sothat;
      let focus = {
        focusId: this.focus.focusId,
        destination: {
          detail: this.focus.destination.detail,
          text1: updatedFocus.ishall,
          text2: updatedFocus.sothat
        }
      };
      this.$store.dispatch("user/updateFocus", focus).then(() => {
        this.$store.dispatch("user/showNotification", {
          title: "Focus Updated",
          description: "Your focus has been successfully updated.",
          variant: "success"
        });
      });
    },
    editCrew(crew) {
      this.focus.crew[crew.index].nickname = crew.nickname;
      this.focus.crew[crew.index].user.email = crew.email;
      this.focus.crew[crew.index].role = crew.role;
      console.log(crew);
      this.show.editPopup = false;
      this.show.overlay = false;
      this.$store
        .dispatch("user/updateFocusCrew", {
          crew: this.focus.crew,
          focus: this.focus
        })
        .then(() => {
          console.log("Updated Crew!");
        });
    },
    deleteCrew(index) {
      console.log(index);
      let crew = this.focus.crew;
      crew.splice(index, 1);
      this.$store
        .dispatch("user/updateFocusCrew", {
          crew: crew,
          focus: this.focus
        })
        .then(() => {
          console.log("Updated Crew!");
        });
    },
    updateCrews(crews) {
      console.log(crews);
      this.focus.crew = crews;
    },
    closeAddCrew(crew) {
      console.log(crew);
      this.show.sidePanel = "DEFAULT";
      this.$store
        .dispatch("user/updateFocusCrew", {
          crew: crew,
          focus: this.focus
        })
        .then(() => {
          console.log("Updated Crew!");
        });
    },
    hideAllPopups() {
      this.show.status_dropdown = false;
    },
    overlayClicked() {
      this.show.status_dropdown = false;
      this.showAddPopup = false;
      this.show.overlay = false;
      this.show.editPopup = false;
      console.log(this.show.overlay);
    },
    toogledropDown() {
      this.show.status_dropdown = !this.show.status_dropdown;
      this.toogleOverlay();
    },
    toogleAddPopup() {
      this.toogleOverlay();
      this.showAddPopup = !this.showAddPopup;
    },
    toogleEditPopup() {
      this.toogleOverlay();
      this.show.editPopup = !this.show.editPopup;
    },
    toogleOverlay() {
      this.show.overlay = !this.show.overlay;
    },
    changeFocusState(state) {
      console.log(state);
      this.overlayClicked();
      this.$store
        .dispatch("user/updateFocusState", {
          focusState: state,
          focus: this.focus
        })
        .then(() => {
          console.log("Updated State!");
        });
    }
  },
  components: {
    UTube,
    AddCrewPopup,
    ToogleButtons,
    Card,
    // VueApexCharts,
    BaseWebComponent,
    DefaultSideBar,
    AddCrewSidebar,
    ToolsSidebar,
    EditFocus,
    AddEngine,
    EditEngine,
    BaseSetup,
    BaseConnect,
    BaseSetupPlayer
  },
  filters: {
    reverse(value) {
      return value.slice().reverse();
    }
  },
  watch: {

  }
};
</script>

<style lang="scss" scoped>
.ico__edit {
  cursor: pointer;
}
.bg-white {
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
}
@media (max-width: 1200px) {
  .bg-white {
    border-top-right-radius: 0 !important;
    border-bottom-right-radius: 20px;
    border-bottom-left-radius: 20px;
  }
}
.bg-grey {
  background: #f3f6f9;
}
.p3040 {
  padding: 30px 40px;
  border-radius: 20px;
}
.p24 {
  padding: 24px;
  border-radius: 20px;
  background-color: #fff;
}
.header-title {
  display: grid;
  grid-template-columns: 1fr 40px;
  .title {
    font-style: normal;
    font-weight: normal;
    font-size: 24px;
    line-height: 33px;
    color: #474d52;
    span {
      color: #8d99a0;
    }
  }
}
.btns-toogle {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  .spot-splash {
    display: flex;
    justify-content: center;
    align-items: center;

    span {
      font-style: normal;
      font-weight: bold;
      font-size: 14px;
      line-height: 19px;
      color: #b3c2cb;
      &.active {
        color: #557efb;
      }
    }
    .switch {
      margin: 0 8px;
      position: relative;
      display: inline-block;
      width: 62px;
      height: 32px;
    }

    .switch input {
      opacity: 0;
      width: 0;
      height: 0;
    }

    .slider {
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: #fff;
      -webkit-transition: 0.4s;
      transition: 0.4s;
    }

    .slider:before {
      position: absolute;
      content: "";
      height: 24px;
      width: 24px;
      left: 4px;
      bottom: 4px;
      background-color: #557efb;
      -webkit-transition: 0.4s;
      transition: 0.4s;
    }

    input:checked + .slider {
      background-color: #fff;
    }

    input:checked + .slider:before {
      -webkit-transform: translateX(26px);
      -ms-transform: translateX(26px);
      transform: translateX(26px);
    }

    /* Rounded sliders */
    .slider.round {
      border-radius: 34px;
    }

    .slider.round:before {
      border-radius: 50%;
    }
  }
}
.card-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-gap: 16px 25px;
  margin-top: 25px;
  @media (max-width: 1200px) {
    grid-template-columns: 1fr 1fr;
  }
  @media (max-width: 560px) {
    grid-template-columns: 1fr;
  }
}
.labels-radial {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  .label-graph {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    .color-label {
      width: 16px;
      height: 16px;
      // background: #52b6d6;
      border-radius: 5px;
      margin: 4px;
    }
  }
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: transparent;
  z-index: 2;
}
@media (max-width: 1200px) {
  .p3040 {
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
  }
  .p24 {
    border-top-left-radius: 0;
    border-top-right-radius: 0;
  }
}
@media (max-width: 568px) {
  .btns-toogle {
    // flex-direction: column;
    // align-items: flex-start;
    justify-content: space-around;
  }
}
@media (max-width: 450px) {
  .btns-toogle {
    height: 160px;
    flex-direction: column;
    align-items: center;
    justify-content: space-around;
  }
}
</style>
