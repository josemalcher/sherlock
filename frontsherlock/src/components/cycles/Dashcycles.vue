<template>
  <div class="dashboard uk-grid">
    <div id="loading" v-if="loading">
       <center><div uk-spinner></div>
       Loading...</center>
     </div>

     <div id="small_loading" v-if="functional_loading">
        <div uk-spinner></div>
        Loading...
      </div>

    <div class="uk-width-4-5">
      <ul uk-tab="animation: uk-animation-slide-left-medium, uk-animation-slide-right-medium" id='cenarios_cases'>
        <li><a @click="cleanCases()">Scenarios</a></li>
        <li><a href="#" v-if="caseslodaded">Cases</a></li>
      </ul>

<ul class="uk-switcher uk-margin" id='cenarios_cases'>
    <li>
      <div class="content scenario">
      <div v-for="scenario in scenarios" :key="scenario.scenario_id" v-if="scenario.cases_stats.total > 0"class="uk-grid ">
          <div @click="fetchCycleCases(scenario.scenario_id)" class="uk-width-4-5" style="cursor: pointer; width: 73% !important;">
            <span>{{ scenario.scenario_name }} <br>
          </span>
            <hr>
          </div>
          <div class="uk-width-1-5" style="width: 27% !important;">
            <ul class="uk-iconnav">
                <li style="padding-left: 8px !important;">
                  <a @click="fetchCycleCases(scenario.scenario_id)" uk-icon="icon: chevron-right;" title="Access Test Cases" uk-tooltip="delay: 300; pos: bottom"></a></li>
                  <li style="padding-left: 8px !important;">
                    <a @click="editScenario(scenario.scenario_name, scenario.scenario_id)" uk-icon="icon: file-edit;" title="Edit Scenario" uk-tooltip="delay: 300; pos: bottom"></a></li>
                <li style="padding-left: 8px !important;">
                  <span class="uk-badge passed">
                    <a style="color: white !important; cursor: default !important;" title="Cases Passed" uk-tooltip="delay: 300; pos: bottom ">
                      {{scenario.cases_stats.total_passed}}
                    </a>
                  </span>
                </li>
                <li style="padding-left: 8px !important;">
                  <span class="uk-badge failed">
                    <a style="color: white !important; cursor: default !important;" title="Cases Failed" uk-tooltip="delay: 300; pos: bottom ">
                      {{scenario.cases_stats.total_error}}
                    </a>
                  </span>
                </li>
                <li style="padding-left: 8px !important;">
                  <span class="uk-badge blocked">
                    <a style="color: white !important; cursor: default !important;" title="Cases Blocked" uk-tooltip="delay: 300; pos: bottom ">
                      {{scenario.cases_stats.total_blocked}}
                    </a>
                  </span>
                </li>
                <li style="padding-left: 8px !important;">
                  <span class="uk-badge">
                    <a style="color: white !important; cursor: default !important;" title="Cases Not Executed" uk-tooltip="delay: 300; pos: bottom ">
                      {{scenario.cases_stats.total_not_executed}}
                    </a>
                  </span>
                </li>
            </ul>
          </div>
      </div>
    </div>
    </li>
    <li>
        <h3 v-if="! caseslodaded">
          Nothing to see here... <br>
          Please click on any scenario to load the cases.
        </h3>
        <div v-else>
          <div class="uk-grid" style="width: 108%;">
            <div class="uk-width-1-1" style="margin-right:20px">
              <h4 class="scenario_name_case">
                Scenario: {{ scenarioFull.scenario_name }} <br>
              </h4>
            </div>
          </div>
          <br>
          <div class="content scenario">
            <transition-group name="fade">
            <div v-for="tstcase in tstcases" :key="tstcase.case_id" class="uk-grid"
              v-if="filter.length === 0 || filter.indexOf(tstcase.case_cycle_state) > -1" >
              <div style="width: 78% !important;" class="uk-width-4-5">
                <span> {{ tstcase.case_name }} </span><br>
                <span> <a uk-icon="icon: tag" style="margin-right:10px"></a>
                  <span v-for="tag in tstcase.tags" class="uk-label"> #{{tag.tag}}</span></span>
              </span>
              <hr>
              </div>
              <div class="uk-width-1-5" style="width: 22% !important;">
                <ul class="uk-iconnav" style="padding: 0px 10px; border: 1px solid;">
                  <li style="margin-right: 10px"> <a title="Edit Case" uk-tooltip="delay: 300; pos: bottom"
                    @click="editCase(tstcase.case_name, scenarioFull.scenario_id, tstcase.case_id)">
                    <span uk-icon="icon: file-edit"></span>
                    </a></li>
                    <li v-bind:class="{passed: tstcase.case_cycle_state === 'passed'}">
                      <center><a title="Pass" uk-tooltip="delay: 300; pos: bottom"
                      @click="changeCaseStatus(tstcase.case_id, scenarioFull.scenario_id, 'passed')">
                      <span v-show="tstcase.case_cycle_state === 'passed'" uk-icon="icon: check" class='passed'></span>
                      <span v-show="tstcase.case_cycle_state !== 'passed'" uk-icon="icon: check"></span>
                    </a></center></li>
                    <li v-bind:class="{failed: tstcase.case_cycle_state === 'error'}">
                      <center><a title="Fail" uk-tooltip="delay: 300; pos: bottom"
                      @click="changeCaseStatus(tstcase.case_id, scenarioFull.scenario_id, 'error')">
                      <span v-show="tstcase.case_cycle_state === 'error'" uk-icon="icon: ban" class='failed'></span>
                      <span v-show="tstcase.case_cycle_state !== 'error'" uk-icon="icon: ban"></span>
                    </a></center></li>
                    <li v-bind:class="{blocked: tstcase.case_cycle_state === 'blocked'}">
                      <center><a title="Block" uk-tooltip="delay: 300; pos: bottom"
                      @click="changeCaseStatus(tstcase.case_id, scenarioFull.scenario_id, 'blocked')">
                      <span v-show="tstcase.case_cycle_state === 'blocked'" uk-icon="icon: lock" class='blocked'></span>
                      <span v-show="tstcase.case_cycle_state !== 'blocked'" uk-icon="icon: lock"></span>
                    </a></center></li>
                    <li>
                      <center><a title="Reset Status" uk-tooltip="delay: 300; pos: bottom"
                      @click="changeCaseStatus(tstcase.case_id, scenarioFull.scenario_id, 'not_executed')">
                      <span uk-icon="icon: reply"></span>
                    </a></center>
                  </li>
                </ul>
              </div>
            </div>
          </transition-group>
          <transition name="fade">
          <div v-if="showNoCase">
            <h3> No Test Cases to be displayed with the selected filters </h3>
          </div>
        </transition>
          </div>
      </div>
    </li>
</ul>

</div>
    <div class="uk-width-1-5">
      <div class="info" style="z-index: 980;" uk-sticky="bottom: true">
        <br><br><router-link class="uk-button uk-button-default" style='margin-top:10px;' :to="{ path: '/project/view/'+ this.projectId }">Return to DashBoard</router-link>
        <br>
        <hr>
          <div v-if="caseslodaded">
            <form>
              <fieldset>
              <legend> Filter by State </legend>
              <center>
                <ul class="uk-iconnav" style="float:none; width: 120px;">
                <li>
                  <a @click="filterPassed = !filterPassed, filterCases('passed')"
                   title="Filter by Passed Cases" uk-tooltip="delay: 300; pos: bottom">
                   <span v-show="filterPassed === false" uk-icon="icon: check"></span>
                   <span v-show="filterPassed === true" uk-icon="icon: check" class="caseFilter"></span>
                  </a></li>
                <li>
                    <a @click="filterFailed = !filterFailed, filterCases('error')"
                    title="Filter by Error Cases" uk-tooltip="delay: 300; pos: bottom">
                  <span v-show="filterFailed === false" uk-icon="icon: ban"></span>
                  <span v-show="filterFailed === true" uk-icon="icon: ban" class="caseFilter"></span>
                </a></li>
                <li>
                  <a @click="filterBlocked = !filterBlocked, filterCases('blocked')"
                  title="Filter by Blocked Cases" uk-tooltip="delay: 300; pos: bottom">
                  <span v-show="filterBlocked === false" uk-icon="icon: lock"></span>
                  <span v-show="filterBlocked === true" uk-icon="icon: lock" class="caseFilter"></span>
                </a></li>
                <li>
                  <a @click="filterNotExecuted = !filterNotExecuted, filterCases('not_executed')"
                  title="Filter by Not Executed Cases" uk-tooltip="delay: 300; pos: bottom">
                  <span v-show="filterNotExecuted === false" uk-icon="icon: reply"></span>
                  <span v-show="filterNotExecuted === true" uk-icon="icon: reply" class="caseFilter"></span>
                </a>
              </li>
            </ul>
          </center>
          </fieldset>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import UIkit from 'uikit'

export default {
  name: 'dashboard',
  data () {
    return {
      projectId: this.$route.params.projectId,
      cycleId: this.$route.params.cycleId,
      scenarios: [],
      scenarioFull: '',
      tstcases: [],
      newScenario: '',
      newCase: '',
      loading: false,
      functional_loading: false,
      caseslodaded: false,
      viewcase: false,
      showNoCase: false,
      filterPassed: false,
      filterBlocked: false,
      filterNotExecuted: false,
      filterFailed: false,
      filter: []
    }
  },
  methods: {
    fetchCycleScenarios () {
      this.$http.get('projects/' + this.projectId + '/cycle/get_scenarios_for_cyle/' + this.cycleId).then(function (response) {
        this.loading = false
        this.scenarios = response.body
        this.scenarios = this.scenarios.reverse()
      })
    },
    changeCaseStatus (caseId, scenarioId, status) {
      this.functional_loading = true
      this.$http.post('projects/' + this.projectId + '/cycle/change_case_state_code', {'case_id': caseId, 'cycle_id': this.cycleId, 'action': status})
      .then(function (response) {
        this.functional_loading = false
        this.fetchCycleCases(scenarioId)
      })
    },
    fetchCycleCases (scenarioId) {
      this.$http.get('projects/' + this.projectId + '/cycle/get_cases_for_cyle/' + this.cycleId + '/scenario/' + scenarioId).then(function (response) {
        this.loading = false
        this.caseslodaded = true
        this.scenarioFull = response.body
        this.tstcases = this.scenarioFull.cases.reverse()
        UIkit.tab('#cenarios_cases', {'animation': 'uk-animation-middle-left'}).show(1)
      })
    },
    editScenario (scenarioName, scenarioId) {
      var vueInstance = this
      UIkit.modal.prompt('Edit Scenario:', scenarioName).then(function (newScenarioName) {
        if (newScenarioName === '') {
          UIkit.notification('<span uk-icon="icon: ban"></span> Scenario can`t be blank', {timeout: '700'})
          return
        } else if (newScenarioName === null) {
          return
        }
        vueInstance.$http.post('scenario/edit', {'scenario_id': scenarioId, 'scenario_name': newScenarioName})
        .then(function (response) {
          UIkit.notification('<span uk-icon="icon: check"></span> Scenario Edited', {timeout: '700'})
          this.fetchCycleScenarios()
        })
      })
    },
    editCase (caseName, scenarioId, caseId) {
      var vueInstance = this
      UIkit.modal.prompt('Edit Test Case:', caseName).then(function (caseName) {
        if (caseName === '') {
          UIkit.notification('<span uk-icon="icon: ban"></span>Test Case can`t be blank', {timeout: '700'})
          return
        } else if (caseName === null) {
          return
        }
        vueInstance.$http.post('scenarios/' + scenarioId + '/tst_case/edit', {'case_id': caseId, 'case_name': caseName})
        .then(function (response) {
          UIkit.notification('<span uk-icon="icon: check"></span> Test Case Edited', {timeout: '700'})
          this.fetchCycleCases(this.scenarioFull.scenario_id)
        })
      }, function () {
        return
      })
    },
    filterCases: function (item) {
      if (this.filter.indexOf(item) > -1) {
        var index = this.filter.indexOf(item)
        this.filter.splice(index, 1)
      } else {
        this.filter.push(item)
      }
      for (var i = 0; i < this.tstcases.length > 0; i++) {
        if (this.filter.length === 0 || this.filter.indexOf(this.tstcases[i].case_cycle_state) > -1) {
          this.showNoCase = false
          return
        }
      }
      this.showNoCase = true
    },
    cleanCases () {
      this.caseslodaded = false
      this.scenarioFull = []
      this.tstcases = []
    }
  },
  created: function () {
    this.loading = true
    this.fetchCycleScenarios()

    this.interval = setInterval(function () {
      this.fetchCycleScenarios()
    }.bind(this), 2000)
  },
  updated: function () {
  },
  beforeDestroy: function () {
    clearInterval(this.interval)
  }
}
</script>

<style scoped>

.uk-iconnav>li{
  padding-left: 0px;
  width: 30px;
}
.dashboard{
  padding: 15px;
}

.uk-label{
  font-size: 12px;
  padding: 0 10px;
  margin: 0 3px;
}

.content{
  min-height: 400px;
  overflow-x: hidden;
  padding: 10px;
  width: 100%;
}

.info{
  width:90%;
  margin-top: 10px;
}

ol, ul {
    padding-left: 5px;
    list-style: none;
}

ul li {
  padding-bottom: 10px;
  padding-top: 10px;
}

.uk-tab>*>a {
    text-transform: capitalize;
    font-size: 25px;
}

.uk-tab>.uk-active>a {
    border-color: #ffa500;
    font-size: 25px;
  }

.divider_new_scenario{
  border-top: 1px solid orange;
  opacity: 0.5;
  width: 50%;
}

.uk-iconnav{
  float: right;
}

.scenario_name_case{
  border: 1px solid orange;
  text-align: center;
  padding: 10px;
}

.passed {
  background: green;
  color: white !important;
  stroke: white !important;
}

.failed {
  background-color: #e80303;
  color: white !important;
  stroke: white !important;
}

.blocked {
  background-color: orange;
  color: whitesmoke !important;
  stroke: whitesmoke !important;
}

.caseFilter{
  background-color: #111;
  color: whitesmoke !important;
  stroke: whitesmoke !important;
}
</style>
