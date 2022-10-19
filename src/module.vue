<template>
	<private-view :title="isMobile ?(new Date(this.dateUnformated).toLocaleDateString('en-us', { weekday:'short',  month:'short', day:'numeric'})):(new Date(this.dateUnformated).toLocaleDateString('en-us', { weekday:'long', year:'numeric', month:'short', day:'numeric'}))" :key="Date.now()">
		<template #title-outer:prepend>
				<v-button class="header-icon"  rounded icon secondary disabled>
					<v-icon name="label" color="rgb(79,84,100)"  />
				</v-button>
			</template>
    <template #navigation>
        <CustomNavigationLeftSidebar />
        </template>
    <template #sidebar>
      <sidebar-detail-group>
        <sidebar-detail title="Views" icon="layers">
          <v-select
            v-model="view"
            :items="views"
            @change="consoleEvent(view)"
          />
        </sidebar-detail>
      </sidebar-detail-group>
      
			
    </template>
    <template #actions>

      
    
      
      <v-button v-if="view == 'weekly'" icon rounded secondary @click="prev"  >
        <v-icon  class="grey--text" name="navigate_before" />
      </v-button>
      <v-button v-if="view == 'weekly'" icon rounded secondary @click="next"  >
        <v-icon  class="grey--text" name="navigate_next" />
      </v-button>
      <v-button  v-if="view == 'weekly'"
        secondary
        class="grey--text today-btn"
        @click="setToday"
      >
        Today
      </v-button>
      <v-menu
        v-if="view == 'weekly'"
        ref="dateUnformated"
        :close-on-content-click="false"
        :show-arrow="true"
        placement="bottom-start"
        seamless
        full-height
      >
        <template #activator="{ toggle }">
          <v-button 
          secondary
          icon rounded
          class="grey--text"
          @click="toggle" 
        >
          <v-icon class="preview" name="event"  />
        </v-button>
          
        </template>
        <div class="date-input calendershow">
          <v-date-picker
            type="date"
            :model-value="(new Date((new Date(this.dateUnformated).getTime()) - (new Date(this.dateUnformated).getTimezoneOffset()) * 60000).toISOString().substr(0, 10))"
            @update:model-value="setDate"
            
          />
        </div>
      </v-menu>
      <v-button  v-if="view == 'table' && filterFlag == false"
        icon rounded secondary
        class="grey--text"
        @click="filterFlag = true"
      >
        <v-icon  class="grey--text" name="filter_list" />
      </v-button>
      <v-select
        v-if="view == 'table' && filterFlag"
        v-model="filter"
        class="filter-field"
        :items="[
          {
						text: 'Id',
						value: 'id'
					},
          {
						text: 'Date',
						value: 'date'
					},
					{
						text: 'Project',
						value: 'project'
					},
					{
						text: 'Department',
						value: 'department'
					},
					{
						text: 'Hours',
						value: 'hours'
					}
				]"
      />
      <v-select
        v-if="view == 'table' && filterFlag"
        v-model="compare"
        class="filter-field"
        :items="[
          {
						text: 'Equals',
						value: '_eq'
					},
          {
						text: `Doesn't equal`,
						value: '_neq'
					},
					{
						text: 'Less than',
						value: '_lt'
					},
					{
						text: 'Less than or equal to',
						value: '_lte'
					},
					{
						text: 'Greater than',
						value: '_gt'
					},
					{
						text: 'Greater than or equal to',
						value: '_gte'
					},
					{
						text: 'Contains',
						value: '_contains'
					},
					{
						text: `Doesn't contain`,
						value: '_ncontains'
					}
				]"
      />
      <v-input
        v-if="view == 'table' && filterFlag"
        class="filter-field"
        v-model="filterContent" 
      />
      <v-button v-if="view == 'table' && filterFlag" icon rounded  @click="getTimesheet" >
        <v-icon  class="grey--text" name="check" />
      </v-button>
      <v-button v-if="view == 'table' && filterFlag" icon rounded secondary @click="clearFilter" >
        <v-icon  class="grey--text" name="close" />
      </v-button>
      <v-button icon rounded @click="dialog=true" >
        <v-icon  class="grey--text" name="add" />
      </v-button>
    </template>
		<template v-if="view =='weekly'">
      
		<v-tabs v-model="tab" :key="Date.now()">
			<v-tab v-for="item in tabItems" :key="item.id" :style="[item.tab == 'Total' ? 'pointer-events: none;':'']" >
        <strong>{{isMobile && item.tab != 'Total' ?item.tab.substring(0, 1):item.tab}}</strong>
        <span class="grey--text text--lighten-1">{{item.hours}}</span>
      </v-tab>
		</v-tabs>

		<v-tabs-items v-model="tab" :key="Date.now()" >
			<v-tab-item v-for="itemTitle in tabItems" :key="itemTitle.id" >
				<v-divider></v-divider>
          <v-list three-line class="d-flex align-center" v-if="itemTitle.tasks.length == 0"  height="200">
            <v-list-item>
              <v-card style="display:flex;align-items:center;justify-content:center;min-height:200px;text-align:center">
                <v-card-title>
                  "If You Can't Measure It, You Can't Improve It" <br> ....Time
                </v-card-title>
              </v-card>
            </v-list-item>
          
          </v-list>
          
            <v-list class="py-0 teb-contents" v-else  three-line v-for="(item, index) in itemTitle.tasks"  :key="index">
              
              <v-list-item  :key="item.project">
                <v-card>
                  <div>
                    <v-card-title>
                      {{item.project}} &mdash;
                      <span class="text-caption text-sm-subtitle-2 font-weight-regular">{{item.department}}</span>
                    </v-card-title>
                    <v-card-text>{{item.notes}}</v-card-text>
                  </div>
                  
                  
                  <v-card-actions>
                    <v-card-subtitle style="margin-top:0">{{item.hours}} </v-card-subtitle>
                    <v-button secondary icon rounded @click="dialogUpdate(item.id)"><v-icon name="edit" /> </v-button>
                  </v-card-actions>
                </v-card>
              </v-list-item>
             
            </v-list>
            
			</v-tab-item>
			
		</v-tabs-items>
    
    
		</template>
		<template v-else>
			<v-table
        allowHeaderReorder
				:headers="[
          {
						text: 'Id',
						value: 'id'
					},
          {
						text: 'Date',
						value: 'date'
					},
					{
						text: 'Project',
						value: 'project'
					},
					{
						text: 'Department',
						value: 'department'
					},
					{
						text: 'Hours',
						value: 'hours'
					},
          {
						text: 'Notes',
						value: 'notes'
					}
				]"
				:items="timesheetTable"
        @click:row="itemClick"
			/>
		</template>
    <v-drawer v-model="dialog" title="Add/Update Task" icon="task" id="task-drawer">
      <form>
        <v-info  v-if="error" icon="error" type="danger"><span style="text-align: left;" v-html="error" /></v-info>
   
        <div class="field full">
          <v-text-overflow text="Project"></v-text-overflow>
          <v-select
            label="Project"
            filled
            v-model="task.project"
            :items="projects"
          />
        </div>
        <div class="field full">
          <v-text-overflow text="Department"></v-text-overflow>
          <v-select
            label="Department"
            filled
            v-model="task.department"
            :items="departments"
          />
        </div>
        <div class="field full">
          <v-text-overflow text="Time"></v-text-overflow>
          <v-input 
            label="Time"
            placeholder="00:00"
            filled
            v-model="task.hours"
            @focusout="timeConvert()"
            id="taskTime"
          />
        </div>
        <div class="field full">
          <v-text-overflow text="Notes"></v-text-overflow>
          <v-textarea v-model="task.notes" />
        </div>
        <div class="field full">
          <v-text-overflow text="Date"></v-text-overflow>
          <div class="task-date-wrapper">
            <v-input 
              type="text"
              style="width:90%"
              v-model="date"
              
              placeholder="--"
            />
            <v-menu
              ref="date"
              :close-on-content-click="false"
              :show-arrow="true"
              placement="bottom-start"
              seamless
              full-height
            >
              <template #activator="{ toggle }">
                <v-icon class="preview date-btn" name="event" @click="toggle" />
              </template>
              <div class="date-input">
                <v-date-picker
                  type="date"
                  :model-value="date"
                  @update:model-value="dateUpdate"
                  style="min-width:300px"
                  
                />
              </div>
            </v-menu>
          </div>
        </div>
                       
        <v-button v-if="!edit" class="field full" @click="dialogSubmit">Save</v-button>
        <v-button v-if="edit" class="field full" @click="updateTask(task.id)">Update</v-button>
        <v-button secondary class="field full close-btn" @click="dialogClose" >Close</v-button>
        <v-button outlined v-if="edit" class="field full delete-btn" @click="deleteItem(task.id)">Delete</v-button>
      </form>
    </v-drawer>
    <v-dialog v-model="dialogDelete" max-width="500px">
      <v-sheet>
        <h2 style="text-align:center">Are you sure you want to delete this item?</h2>
        <v-button secondary class="field full" @click="closeDelete">Cancel</v-button>
        <v-button class="field full" @click="deleteTask">OK</v-button>
      </v-sheet>
    </v-dialog>
		<!-- <v-button v-on:click="logToConsole">Log collections to console</v-button> -->
	</private-view>
</template>

<script>
import moment from 'moment';
// import { useApi, useStores } from '@directus/extensions-sdk';
export default {
	// setup() {
	// 	const api = useApi();

	// 	const { useCollectionsStore } = useStores();
	// 	const collectionsStore = useCollectionsStore();

	// 	console.log(collectionsStore)
	// },
	data() {
		return {
      toggle:'',
			collections: null,
			activePage:'',
			dialogm1: '',
    dialog: false,
    absolute: true,
    opacity: 1,
    overlay: false,
    drawer: null,
    tab: [0],
    tabItems: [],
    tabContentItems: [],
    links: [
      'Time',
      'Projects',
      'Team',
      'Reports',
    ],
    projects: [],
    departments: [],
    dateUnformated : new Date().toString(),
    dateShow : new Date().toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"}),
    calendarShow : '',
    task: {
      id: Date.now() + parseInt(Math.random()*100),
      userId:'',
      date: new Date(),
      dateTime: new Date().getTime(),
      project:'',
      department:'',
      hours:'',
      notes:'',
      status:'published'
    },
    date: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
    menu: false,
    modal: false,
    menu2: false,
    edit: false,
    deleteId:'',
    dialogDelete :false,
    valid:false,
    snackbar: false,
    snackbarText: '',
    start:"",
    end:"",
    team:'',
		views:[
			{
				text: 'Weekly',
				value: 'weekly',
			},
			{
				text: 'Table',
				value: 'table',
			},
		],
		view : 'weekly',
    isMobile : false,
    user:[],
    error:'',
    timesheetTable:[],
    filter:'id',
    search:'',
    compare:'_eq',
    filterContent:'',
    filterFlag : false,
		
		};
	},
	inject: ['api'],
	mounted() {
		// log the system field so you can see what attributes are available under it
		// remove this line when you're done.
    
    if(window.innerWidth <768){
      this.isMobile = true;
    }
    else{
      this.isMobile = false;
    }
    window.addEventListener('resize', this.handleResize)
		console.log(this.api);
    this.getTimesheet();
		this.setTabContentItems();
    this.projectList();
    this.departmentList();

    this.api.get('/users/me').then(res => {
      // console.log(res);
      this.user = res.data.data;
    });



    
    
	},
  destroyed() {
    window.removeEventListener('resize', this.handleResize)
  },
	methods: {
    consoleEvent(e){
      console.log(e)
    },
		logToConsole: function () {
			console.log(this.collections);
		},
    dateUpdate(e){
      this.date = e;
    },
    handleResize (e){
      // console.log(e.target.innerWidth);
      if(e.target.innerWidth <768){
        this.isMobile = true;
      }
      else{
        this.isMobile = false;
      }
    },
		setDate (x) {
      // this.dateUnformated = moment(x).format('MMMM Do YYYY, h:mm:ss a');
      // console.log("moment" + this.dateUnformated);
      this.dateUnformated = new Date((new Date(x).getTime()) + (new Date(x).getTimezoneOffset()) * 60000);
      // console.log("date" + this.dateUnformated);
      // this.dateShow = new Date(x).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
      // this.calendarShow = !this.calendarShow
    },

    setToday () {
      this.dateUnformated = new Date().toString();
      // this.dateShow = new Date().toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"})
    },
    prev () {
      let nextDate = new Date(this.dateUnformated);
      nextDate.setDate(nextDate.getDate() - 1);
      //console.log(nextDate);
      
      this.dateUnformated = nextDate.toString();
      // this.dateShow = nextDate.toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
    },
    next () {
      let nextDate = new Date(this.dateUnformated);
      nextDate.setDate(nextDate.getDate() + 1);
      //console.log(nextDate);
      this.dateUnformated = new Date(nextDate).toString();
      // this.dateShow = nextDate.toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
    },
    timestrToSec(timestr) {
      // console.log(timestr)
      var parts = timestr.split(":");
      return (parts[0] * 3600) +
            (parts[1] * 60) 
    },
    pad(num) {
      if(num < 10) {
        return "0" + num;
      } else {
        return "" + num;
      }
    },
    formatTime(seconds) {
      // console.log(seconds);
      return [this.pad(Math.floor(seconds/3600)),
              this.pad(Math.floor(seconds/60)%60)
              ].join(":");
    },
    hourCalculator(tabitem){
      // console.log(tabitem);
      for(let i=0;i< this.tabContentItems.length; i++){
        // console.log(this.tabContentItems[i]);
        if(this.tabContentItems[i].day == tabitem.tab || tabitem.tab == 'Total'){
          tabitem.hours = this.formatTime(this.timestrToSec(tabitem.hours) + this.timestrToSec(this.tabContentItems[i].hours));
        }
      }
      this.tabItems[tabitem.id].hours = tabitem.hours;
      return tabitem.hours
    
    },
    getTimesheet(){
      let url = `/items/Timesheet`;
      if(this.filterContent !== ''){
        url += `?filter={"_and":[{"${this.filter}":{"${this.compare}":"${this.filterContent}"}}]}`
      }
      this.api.get(url).then(res => {
        this.timesheetTable=[];
        if(res.data.data.length > 0) {
          try {
              for (const doc of res.data.data) {
                // console.log(doc);
                this.timesheetTable.push(doc);
              }
              
            } catch (err) {}
        }
      })
    },
    itemClick(res){
      this.task = res.item;
      this.edit = true;
      this.dialog = true;
    },
    clearFilter(){
      this.filter = 'id';
      this.compare = '_eq';
      this.filterContent = '';
      this.filterFlag = false;
      this.getTimesheet();
    },

    
    setTabContentItems(){
      let that = this;
      let currDate = new Date(this.dateUnformated);
      // console.log(currDate.getTime(),moment(currDate).weekday(0).get('date'));
      let startDate = moment(currDate).weekday(0);
      currDate.setDate(startDate.get('date'));
      currDate.setMonth(startDate.get('month'));
      currDate.setFullYear(startDate.get('year'));

      // currDate.setDate(currDate.getDate() - currDate.getDay() +1);
      currDate.setHours("00");
      currDate.setMinutes("00");
      currDate.setSeconds("00");
      this.start = parseFloat(new Date((currDate.getTime()) + currDate.getTimezoneOffset() * 60000).getTime());
      // console.log(currDate);
      currDate.setHours("23");
      currDate.setMinutes("59");
      currDate.setSeconds("59");
      currDate.setDate(currDate.getDate() +6);
      // console.log(currDate);
      this.end = parseFloat(new Date((currDate.getTime()) + currDate.getTimezoneOffset() * 60000).getTime());
      // console.log(this.start,this.end);
			this.api.get(`/items/Timesheet?filter={"_and":[{"dateTime":{"_lte":"${this.end}"}},{"dateTime":{"_gte":"${this.start}"}}]}`).then((res) => {
				this.collections = res.data.data;
				
        if(res.data.data.length > 0) {
          try {
              that.tabContentItems = [];
              for (const doc of res.data.data) {
                // console.log(doc);
                that.tabContentItems.push(doc);
              }
              // console.log(that.tabContentItems);
              that.setTabItems();
            } catch (err) {}
        }
        else{
          that.setTabItems();
        }
        // console.log(that.tabContentItems);
			});

     


    },
		setTabItems(){
      let curr = new Date(this.dateUnformated);
      let currTime = parseFloat(new Date((curr.getTime()) + curr.getTimezoneOffset() * 60000).getTime());
      // console.log(curr,currTime);
      if(this.start<= currTime && currTime <= this.end){
      let currId = '',
      tabItems = [],
      totalItem = {
        id:"w7"+Math.floor(Math.random() * 1000),
        tab:"Total",
        tasks:[],
        hours:"00:00"
        };
      
      let tabDate = new Date(this.start);
      tabDate.setDate(tabDate.getDate());
      tabDate.setHours("00");
      tabDate.setMinutes("00");
      tabDate.setSeconds("00");

      for (let i = 0; i < 7; i++) {
        let item = {};
        // item.id = "w"+i+Math.floor(Math.random() * 1000);
        item.id = i;
        // console.log(tabDate.getDate());
        let first =(i==0 && tabDate.toLocaleDateString('en-us', { weekday:"long" }) == 'Sunday'? tabDate.getDate(): tabDate.getDate()+1) ;
        // console.log(first);
        let day = new Date(tabDate.setDate(first)).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
        if((tabDate.getDate()) == (new Date(this.dateUnformated).getDate())){
          currId = i;
          // console.log(currId);
        }
        item.tab = new Date(day).toLocaleDateString('en-us', { weekday:"short"});
        item.date= new Date(day).toString();
        // console.log(i,"-----",item.date)
        item.tasks = [];
        item.hours = "00:00";
        for(let j=0;j<this.tabContentItems.length;j++){
          // console.log(this.tabContentItems[j].date == day);
          if(this.tabContentItems[j].date == day){
            item.tasks.push(this.tabContentItems[j]);
            item.hours = this.formatTime(this.timestrToSec(item.hours) + this.timestrToSec(this.tabContentItems[j].hours));
            totalItem.tasks.push(this.tabContentItems[j]);
            totalItem.hours = this.formatTime(this.timestrToSec(totalItem.hours) + this.timestrToSec(this.tabContentItems[j].hours));
            // console.log(totalItem.hours);
          }
        }
        // console.log(item);

       
        tabItems.push(item);
      }
      // console.log(totalItem);
      tabItems.push(totalItem);
      this.tabItems = tabItems;
      this.tab = [currId];
      // console.log("tab ----- ",this.tabItems,this.tab);
    }
    else{
      // console.log("else part");
      this.setTabContentItems();
      
    }
    },
    dialogUpdate(id){
      let tasks = this.tabContentItems.filter((tsk) => (tsk.id == id));
      this.task = tasks[0];
      // console.log(tasks);
      this.edit = true;
      this.dialog = true;
    },
    dialogClose(){
      this.error = '';
      this.setTabContentItems();
      this.getTimesheet();
      this.dialog = false;
      this.edit = false;
      this.date = new Date((new Date(this.dateUnformated).getTime()) - (new Date(this.dateUnformated).getTimezoneOffset()) * 60000).toISOString().substr(0, 10);
      this.task.id='';
      this.task.date = new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
      this.task.dateTime = new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).getTime();
      this.task.project='';
      this.task.department='';
      this.task.hours='';
      this.task.notes=''
    
      // console.log(this.task);
            
    },
    formValidate(){
      this.error = "";
      // console.log(this.task);
      if(this.task.project == ''){
        this.error = this.error + 'Project is required <br>'
      }
      if(this.task.department == ''){
        this.error = this.error + 'Department is required <br>'
      }
      if(this.task.hours == '' || this.task.hours == null ){
        this.error = this.error + 'Hours is required <br>'
      }
      if(this.error != ''){
         console.log(this.error);
        return false
      }
      else{
        // console.log('no error');
        return true
      }
    },
    dialogSubmit(){
      if(this.formValidate()){

        let that = this
      // if (this.$refs.form.validate()) {
        this.task.id = Date.now() + parseInt(Math.random()*100);
        // this.task.userId = this.$auth.user.id;
        this.task.date = String(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"}));
        this.task.dateTime = parseFloat(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).getTime());
        
        this.api.post(`/items/Timesheet`,
        {
          date : this.task.date,
          dateTime : this.task.dateTime,
          department : this.task.department,
          hours : this.task.hours,
          notes : this.task.notes,
          project : this.task.project,
          status: "published"
        }).then(function (response) {
            // console.log(response);
            that.dateUnformated = new Date((new Date(that.date).getTime()) + (new Date(that.date).getTimezoneOffset()) * 60000);
            that.dialogClose();
          })
          .catch(function (error) {
            console.log(error);
          });

      

      }
      



      
    },
    getTask(id){
      
    },
    async updateTask(id){
      if(this.formValidate()){
        
        this.task.date = String(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"}));
        this.task.dateTime = parseFloat(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).getTime());

        let that = this
        

        
        // console.log(this.task);

        this.api.patch(`/items/Timesheet/${this.task.id}`,
    {
      date : this.task.date,
      dateTime : this.task.dateTime,
      department : this.task.department,
      hours : this.task.hours,
      notes : this.task.notes,
      project : this.task.project,
      status: "published",
      userId : this.task.userId
    }).then(function (response) {
        // console.log(response);
        that.dateUnformated = new Date((new Date(that.date).getTime()) + (new Date(that.date).getTimezoneOffset()) * 60000);
        that.dialogClose();
      })
      .catch(function (error) {
        console.log(error);
      });

     

      }
    },
    async deleteTask(){
      let id = this.deleteId;
      let that = this
      this.api.delete(`/items/Timesheet/${this.task.id}`).then(function (response) {
        // console.log(response);
        that.dateUnformated = new Date((new Date(that.date).getTime()) + (new Date(that.date).getTimezoneOffset()) * 60000);
        that.dialogDelete = false;
        that.dialogClose();
      })
      
    },
    tabChange (id) {
      // console.log("hi");
      let currTab = this.tabItems.filter((tab) => (tab.id == id))[0] ;
      if(currTab.date != null){
        this.dateUnformated = new Date(currTab.date).toString();
        // console.log(currTab,new Date(currTab.date).toString());
      }
    },
    projectList() {
      
      let that = this;

      

      this.api.get(`/items/project`)
      .then(function (res) {
        that.projects = [];
        if(res.data.data.length > 0) {
          try {
            for (const doc of res.data.data) {
              let x = {
                        text: doc.pcode +' - '+ doc.pname,
                        value: doc.pcode +' - '+ doc.pname,
                      }
              
              that.projects.push(x);
            }
            
          } catch (err) {}
        }
        // console.log(that.projects);
      })
      .catch(function (error) {
        console.log(error);
      });
    },
    departmentList() {
      let that = this;
      this.api.get(`/items/department`)
      .then(function (res) {
        that.departments = [];
        if(res.data.data.length > 0) {
          try {
            for (const doc of res.data.data) {
              let x = {
                        text: doc.dname,
                        value: doc.dname,
                      }
              
              that.departments.push(x);
            }
            
          } catch (err) {}
        }
        
      })
      .catch(function (error) {
        console.log(error);
      });

    },
    deleteItem(id) {
      this.dialogDelete = true;
      this.deleteId = id;
    },
    closeDelete() {
      this.deleteId = '';
      this.dialogDelete = false
    },
    timeConvert(){ 
      let number = this.task.hours;
      // console.log(number);
      if(number == '' || number == null){
        this.error = "Hours is Required "
      }
      else{
      if(number.includes(':')){
        // console.log(number);
       this.task.hours = number; 
      }
      else{
        // console.log(number);
        // Check sign of given number
      var sign = (number >= 0) ? 1 : -1;

      // Set positive value of number of sign negative
      number = number * sign;

      // Separate the int from the decimal part
      var hour = Math.floor(number);
      var decpart = number - hour;

      var min = 1 / 60;
      // Round to nearest minute
      decpart = min * Math.round(decpart / min);

      var minute = Math.floor(decpart * 60) + '';

      // Add padding if need
      // if (minute.length < 2) {
      // minute = '0' + minute; 
      // }

      // Add Sign in final result
      sign = sign == 1 ? '' : '-';

      // Concate hours and minutes
      this.task.hours = sign + this.pad(hour) + ':' + this.pad(minute);
    
        }
    }
    },
  },
  watch: {
    dateUnformated: function (val, oldVal) {
      // console.log(val);
      this.setTabItems();
      this.date = new Date((new Date(val).getTime()) - (new Date(val).getTimezoneOffset()) * 60000).toISOString().substr(0, 10);
      // this.dateShow = new Date(val).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
      // console.log(this.date,this.tabItems);
      // console.log("hello",val);

    },
    tab: function (val, oldVal) {
      this.tabChange(val)
    },
    dialog: function (val, oldVal){
      if(!val){
        this.dialogClose();
      }
      
    }
  },
	
};
</script>
<style scoped>
.v-card {
  min-width: 100%;
  max-width: 100%;
}
.v-tabs.horizontal {
    justify-content: space-between;
    display: flex;
    flex-direction: row;
}
.field.full {
  padding: 10px 20px;
}
.v-text-overflow{
  padding-bottom: 10px;
}

.v-tab.horizontal {
  flex-direction: column;
  height: 60px !important;
  border-bottom: 3px solid transparent !important;
}
.v-tab.horizontal.active {
  border-bottom: 3px solid #5842d0 !important;
  font-weight: 800;
}

.v-card-subtitle{
  margin-top: 0px;
  padding-bottom: 0;
  padding-right: 70px;
}
.v-card-actions {
    align-items: center;
}
.calendershow {
    width: 300px;
}
.date-btn{
  position: absolute;
  top: 25%;
  right: 20px;
}
.task-date-wrapper{
  position: relative;
}
.v-card:hover {
    background-color: #f0f4f9;
}

@media screen and (max-width:767px) {
  .v-tabs.horizontal{
    
    flex-wrap: wrap;
  }
  .v-tab.horizontal {
    padding: 10px 5px !important;
    min-height: 50px;
  }
  .v-card-subtitle{
    padding: 0 5px;
  }
  .v-tabs.horizontal .v-tab:last-child{
    background: var(--v-button-background-color-disabled);
    flex-direction: row;
    gap: 10px;
    flex: 0 0 100%;
    max-width: 100%;
  }
  .v-card{
    display: flex;
    justify-content: space-between;
  }
  .v-card-actions {
    padding: 10px;
}
  .today-btn > button {
    min-width: min-content !important;
    padding: 10px !important;
  }
  .actions .action-buttons>*:not(:last-child) {
    margin-right: 5px;
  }
  .v-item-group.v-tabs-items, .v-tabs, .v-table {
    padding: 0;
  }
}
@media screen and (min-width:768px) {
  .v-item-group.v-tabs-items, .v-tabs, .v-table {
    padding: 0 32px;
  }
  .v-card {
    display: flex;
    justify-content: space-between;
  }
  .v-card:first-child{
    flex: 0 0 80%;
    max-width: 80%;
  }
  .field.full.close-btn{
    padding-left: 0;
  }
  .field.full.delete-btn{
    float: right;
  }
  .filter-field{
    min-width: 160px;
  }
}

</style>
<style>
.v-tab-item .v-list:nth-child(2) {
    padding-top: 8px;
}
.v-drawer .cancel,.header-bar .nav-toggle {
    display: none;
}
.v-tab.horizontal.active strong{
  font-weight: 800;
}
.v-info[data-v-130d5c84] {
    flex-direction: revert;
    padding: 10px 25px;
    gap: 20px;
}
.icon[data-v-130d5c84] {
    width: 20px;
    height: 20px;
    margin-top: 5px;
}
.title[data-v-130d5c84]{
    font-size: 15px;
    font-weight: 600;
}
.content[data-v-130d5c84]{
  text-align: left;
}
.v-list-item {
    padding: 0 !important;
}
@media screen and (max-width:767px) {
  .today-btn > button {
    min-width: min-content !important;
    padding: 10px !important;
  }
  .field.full ,.field.full button{
  width: 100%;
  }
  #dialog-outlet .container.right .v-drawer{
    max-width: 85vw;
  }
  .header-bar .title-container[data-v-757091cf]{
    margin-left: 0;
  }
  
}
</style>
